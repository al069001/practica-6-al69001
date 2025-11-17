# Modelado – Generador de Curvas IDF

Este módulo contiene el modelado completo.

## Código completo
```python
"""
Curvas IDF - Generador y Analizador
Archivo: curvas_idf_app.py
Descripción:
  Interfaz PyQt5 para cargar series pluviométricas (timestamp + precip_mm),
  calcular máximos móviles por duración, ajustar Gumbel y generar curvas IDF
  (intensidad vs duración) para varios periodos de retorno.
Requisitos: pandas, numpy, scipy, matplotlib, pyqt5
Uso: python curvas_idf_app.py
"""

import sys
import numpy as np
import pandas as pd
from scipy import stats
from PyQt5.QtWidgets import (
    QApplication, QWidget, QVBoxLayout, QHBoxLayout, QPushButton, QLabel,
    QLineEdit, QFileDialog, QMessageBox, QComboBox, QTableWidget, QTableWidgetItem
)
from PyQt5.QtCore import Qt
from matplotlib.backends.backend_qt5agg import FigureCanvasQTAgg as FigureCanvas
from matplotlib.figure import Figure

def resample_to_min_rule(df):
    diffs = df.index.to_series().diff().dropna()
    mode = diffs.mode()[0]
    if mode < pd.Timedelta('1H'):
        rule = f'{int(mode.total_seconds()/60)}T'
    else:
        rule = f'{int(mode.total_seconds()/3600)}H'
    df_rs = df.resample(rule).sum()
    return df_rs, rule

def compute_moving_max_per_year(df, window_minutes, min_rule='10T'):
    freq_minutes = pd.Timedelta(min_rule).total_seconds()/60
    window_periods = int(round(window_minutes / freq_minutes))
    if window_periods < 1:
        window_periods = 1
    rolling_sum = df.iloc[:,0].rolling(window=window_periods, min_periods=1).sum()
    yearly_max = rolling_sum.resample('A').max().dropna()
    return yearly_max

def fit_gumbel(series):
    data = series.values
    data = data[data > 0]
    if len(data) < 3:
        raise ValueError("Necesitas al menos 3 años de datos positivos para ajustar Gumbel.")
    loc, scale = stats.gumbel_r.fit(data)
    return loc, scale

def gumbel_quantile(loc, scale, T):
    p = 1 - 1.0/T
    return stats.gumbel_r.ppf(p, loc=loc, scale=scale)

class CurvasIDFApp(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Generador de Curvas IDF - Campeche (Demo)")
        self.setGeometry(100,100,1100,700)
        self.df = None
        self.min_rule = None
        self.yearly_cache = {}
        self.idf_result = None
        self._build_ui()

    def _build_ui(self):
        main = QVBoxLayout()
        top = QHBoxLayout()
        self.load_btn = QPushButton("📂 Cargar Excel/CSV")
        self.load_btn.clicked.connect(self.load_file)
        top.addWidget(self.load_btn)
        top.addWidget(QLabel("Duraciones (min, coma):"))
        self.durations_edit = QLineEdit("5,10,15,30,60,120,360,1440")
        self.durations_edit.setFixedWidth(240)
        top.addWidget(self.durations_edit)
        top.addWidget(QLabel("Periodos retorno (años):"))
        self.Ts_edit = QLineEdit("2,5,10,25,50,100")
        self.Ts_edit.setFixedWidth(200)
        top.addWidget(self.Ts_edit)
        top.addWidget(QLabel("Método:"))
        self.method_combo = QComboBox()
        self.method_combo.addItems(["gumbel","empirical"])
        top.addWidget(self.method_combo)
        self.calc_btn = QPushButton("🔢 Calcular IDF")
        self.calc_btn.clicked.connect(self.calculate_idf)
        self.calc_btn.setEnabled(False)
        top.addWidget(self.calc_btn)
        main.addLayout(top)
        mid = QHBoxLayout()
        self.fig = Figure(figsize=(6,5))
        self.canvas = FigureCanvas(self.fig)
        mid.addWidget(self.canvas, 2)
        self.table = QTableWidget()
        self.table.setColumnCount(4)
        self.table.setHorizontalHeaderLabels(["T (años)","Duración (min)","Depth (mm)","Intensity (mm/h)"])
        mid.addWidget(self.table, 1)
        main.addLayout(mid)
        self.log = QLabel("Estado: esperando archivo...")
        main.addWidget(self.log)
        self.setLayout(main)

    def load_file(self):
        path, _ = QFileDialog.getOpenFileName(self, "Abrir serie precipitación", "", "Excel (*.xlsx *.xls);;CSV (*.csv);;All files (*)")
        if not path:
            return
        try:
            if path.lower().endswith('.csv'):
                df = pd.read_csv(path, parse_dates=[0])
            else:
                df = pd.read_excel(path, parse_dates=[0])
            df = df.dropna(axis=0, how='any')
            df = df.iloc[:, :2]
            df.columns = ['timestamp','precip_mm']
            df = df.set_index('timestamp').sort_index()
            self.df, self.min_rule = resample_to_min_rule(df)
            self.log.setText(f"Archivo cargado: {path} | resolución: {self.min_rule} | registros: {len(self.df)}")
            self.calc_btn.setEnabled(True)
            self.yearly_cache = {}
            self.idf_result = None
        except Exception as e:
            QMessageBox.critical(self, "Error lectura", str(e))
            return

    def calculate_idf(self):
        if self.df is None:
            QMessageBox.warning(self, "Sin datos", "Carga primero un archivo con serie de precipitación.")
            return
        try:
            durations = [int(x.strip()) for x in self.durations_edit.text().split(",") if x.strip()]
            Ts = [int(x.strip()) for x in self.Ts_edit.text().split(",") if x.strip()]
        except Exception as e:
            QMessageBox.critical(self, "Error", "Duraciones o periodos de retorno no válidos.")
            return
        method = self.method_combo.currentText()
        yearly = {}
        for d in durations:
            try:
                ym = compute_moving_max_per_year(self.df, d, min_rule=self.min_rule)
                yearly[d] = ym
            except Exception as e:
                yearly[d] = pd.Series([], dtype=float)
        self.yearly_cache = yearly
        idf = {T: {} for T in Ts}
        for d, series in yearly.items():
            if len(series) < 3:
                data = series.values
                for T in Ts:
                    if len(data)>0:
                        p = 1 - 1.0/T
                        depth = np.percentile(data, p*100)
                    else:
                        depth = 0.0
                    idf[T][d] = max(depth, 0.0)
                continue
            if method == 'gumbel':
                try:
                    loc, scale = fit_gumbel(series)
                    for T in Ts:
                        depth = gumbel_quantile(loc, scale, T)
                        idf[T][d] = max(depth, 0.0)
                except Exception as e:
                    data = series.values
                    for T in Ts:
                        p = 1 - 1.0/T
                        depth = np.percentile(data, p*100)
                        idf[T][d] = max(depth, 0.0)
            else:
                data = series.values
                for T in Ts:
                    if len(data)>0:
                        p = 1 - 1.0/T
                        depth = np.percentile(data, p*100)
                    else:
                        depth = 0.0
                    idf[T][d] = max(depth, 0.0)
        self.idf_result = idf
        self.plot_idf(idf, durations, Ts)
        self.populate_table(idf, durations, Ts)
        self.log.setText("Cálculo completado. Revisa la gráfica y la tabla.")

    def plot_idf(self, idf_dict, durations, Ts):
        self.fig.clear()
        ax = self.fig.add_subplot(111)
        for T in Ts:
            ds = sorted(durations)
            intensities = []
            for d in ds:
                depth = idf_dict.get(T, {}).get(d, 0.0)
                hours = d/60.0 if d>0 else 1.0
                intensity = depth / hours if hours>0 else 0.0
                intensities.append(intensity)
            ax.loglog(ds, intensities, marker='o', label=f"T={T}y")
        ax.set_xlabel("Duración (min)")
        ax.set_ylabel("Intensidad (mm/h)")
        ax.set_title("Curvas IDF")
        ax.grid(True, which='both', linestyle='--', linewidth=0.5)
        ax.legend()
        self.canvas.draw()

    def populate_table(self, idf_dict, durations, Ts):
        rows = []
        for T in Ts:
            for d in sorted(durations):
                depth = idf_dict.get(T, {}).get(d, 0.0)
                intensity = (depth / (d/60.0)) if d>0 else 0.0
                rows.append((T,d,round(depth,3),round(intensity,3)))
        self.table.setRowCount(len(rows))
        for i, r in enumerate(rows):
            for j, val in enumerate(r):
                item = QTableWidgetItem(str(val))
                self.table.setItem(i, j, item)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    win = CurvasIDFApp()
    win.show()
    sys.exit(app.exec_())

```
