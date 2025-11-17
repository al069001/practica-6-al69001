# ☀️ Proyecto de Curvas Climáticas – Python + PyQt5 🌧️

Esta aplicación, desarrollada en Python con una interfaz gráfica creada con PyQt5, permite cargar datos climáticos desde un archivo Excel y generar curvas de temperatura, humedad y precipitación. El análisis de datos climáticos es fundamental para comprender patrones meteorológicos, predecir tendencias futuras y tomar decisiones informadas en diversos campos. Esta aplicación facilita este proceso al proporcionar una herramienta interactiva para visualizar y analizar datos climáticos.

**🌟 Componentes Clave:** La aplicación utiliza la librería Pandas para leer y manipular datos climáticos desde un archivo Excel (`datos_clima.xlsx`), que debe contener columnas para Fecha, Temperatura, Humedad y Precipitación. Matplotlib se encarga de generar gráficas de líneas que muestran la evolución de estos parámetros a lo largo del tiempo. PyQt5 proporciona una interfaz gráfica intuitiva que permite cargar archivos y visualizar los datos de forma dinámica.

**🗂️ Archivos Necesarios:**

*   `datos_clima.xlsx`: Archivo Excel con datos climáticos (Fecha, Temperatura, Humedad, Precipitacion).
*   
*   `curvas.py`: (Archivo principal de la aplicación).
*   `requirements.txt`: Lista de dependencias (`PyQt5`, `numpy`, `pandas`, `matplotlib`, `openpyxl`).

**🚀 Características:** Interfaz gráfica limpia y moderna, carga de archivos `.xlsx`, gráficas automáticas con Matplotlib, lectura de datos con pandas, compatible con Windows, Linux y macOS.

**🧪 Requisitos:** Para ejecutar la aplicación, instala las dependencias con: `pip install -r requirements.txt`

**▶️ Cómo ejecutar el programa:**

1.  Asegúrate de tener Python instalado.
2.  Instala las dependencias: `pip install -r requirements.txt`
3.  Ejecuta: `python curvas.py`

**📌 Autor:** Jorge Elias Naal Che - Proyecto Escolar – Análisis Climático - 2025

**✅ Conclusión:** Esta aplicación proporciona una herramienta interactiva y fácil de usar para visualizar y analizar datos climáticos, facilitando la comprensión de patrones meteorológicos y la toma de decisiones informadas.
