# Generador de Curvas IDF – Proyecto en Python

## 📘 Descripción General
Este proyecto implementa una herramienta para **generar, visualizar y exportar curvas IDF (Intensidad–Duración–Frecuencia)** utilizando datos de precipitaciones.  
La aplicación permite cargar datos desde Excel, procesarlos y aplicar ajustes estadísticos para generar curvas utilizadas en ingeniería hidráulica.

---

## 🧩 Fases del Proyecto

### **🔹 Fase 1 — Carga y validación de datos**
- Lee un archivo Excel con series de precipitación.
- Verifica estructura, valores y columnas esperadas.
- Convierte los datos en arreglos manejables para análisis.

### **🔹 Fase 2 — Procesamiento estadístico**
- Calcula intensidades para diferentes duraciones.
- Aplica modelos matemáticos de ajuste (IDF).
- Obtiene parámetros necesarios para la curva.

### **🔹 Fase 3 — Generación de curva IDF**
- Grafica la relación Intensidad–Duración.
- Produce curvas por periodo de retorno.
- Permite visualizar resultados dentro de la interfaz.

### **🔹 Fase 4 — Exportación de resultados**
- Guarda curvas o tablas generadas.
- Permite generar archivos externos para reportes.

---

## 🛠️ Uso del Script
Ejecuta:

```bash
python curvas_idf_app.py
```

El programa cargará el archivo Excel incluido en el proyecto y generará la curva IDF correspondiente.

---

## 📂 Estructura del Proyecto

```
curvas_idf_project/
├─ curvas_idf_app.py        # Código principal
├─ campeche_precip_10min.xlsx   # Base de datos de precipitación
└─ README.md                # Este documento
```

---

## 🧑‍💻 Requerimientos
- Python 3.9 o superior  
- Librerías:
  - pandas  
  - matplotlib  
  - numpy  
  - openpyxl

---

## ✔️ Conclusión
Este proyecto automatiza la creación de curvas IDF a partir de datos reales de precipitación.  
Es una herramienta útil para estudiantes y profesionales en ingeniería hidráulica, hidrología y análisis de riesgo.

