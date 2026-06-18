# Analytics-de-Ensayos-Clinicos-Portal-de-Eficacia-y-Seguridad-Neurologica

Este proyecto implementa un pipeline de datos de extremo a extremo (End-to-End) para evaluar la eficacia y el perfil de seguridad de un fármaco neurológico en una cohorte simulada de **1,000 pacientes** a lo largo de **4 visitas cronológicas**. 

[**Link al Dashboard Interactivo:**](https://public.tableau.com/app/profile/isa.v.zquez.garc.a/viz/PortaldeAnlisisdeEficaciaySeguridadenEnsayosClnicos_/Historia1)

---

## Arquitectura del Proyecto

El proyecto se divide en tres capas de desarrollo:
1. **Almacenamiento (SQL):** Estructuración y persistencia de los datos clínicos mediante SQLite.
2. **Procesamiento y Validación Estadística (Python):** Limpieza, agregación y ejecución de pruebas de hipótesis analíticas (`SciPy`, `Pandas`).
3. **Visualización (Tableau):** Capa con arquitectura *Viz-in-Tooltip* e Historias narrativas.

---

## 1. Modelado de Datos y SQL

Los datos brutos del ensayo clínico se estructuraron en una base de datos relacional utilizando **SQLite**. Se definieron llaves primarias, foráneas y restricciones de integridad para asegurar la calidad de la información médica.

### Query Clave: Preparación del Dataset para Tableau
Para alimentar el dashboard de manera eficiente, se consolidó un *flat table* uniendo la demografía de los pacientes, sus asignaciones de dosis y los reportes de evolución temporal mediante el siguiente enfoque técnico:

```sql
SELECT 
    p.paciente_id,
    p.edad,
    p.genero,
    d.tipo_dosis,
    v.numero_visita,
    v.puntaje_mmse,
    v.reporto_insomnio,
    v.reporto_cefalea
FROM pacientes p
JOIN asignacion_dosis d ON p.paciente_id = d.paciente_id
JOIN visitas_seguimiento v ON p.paciente_id = v.paciente_id
ORDER BY p.paciente_id, v.numero_visita;
```

## 2. Validación Estadística (SciPy)

Para respaldar las visualizaciones del Dashboard, se realizaron dos análisis críticos en Python:
1. **Prueba de Eficacia (ANOVA de una vía):** Se evaluó el puntaje MMSE en la Visita 4. 
   El resultado arrojó un $p < 0.05$, confirmando que la Dosis Alta estabiliza el deterioro cognitivo 
   de forma estadísticamente significativa en comparación con el Placebo.
   
2. **Prueba de Seguridad (Chi-cuadrado):** Se analizó la tasa de deserción por grupo. 
   Con un $p = 0.02$, se rechaza la hipótesis nula, demostrando que la deserción en la Dosis Alta 
   está vinculada a la intolerancia al tratamiento (cefalea/insomnio).
   
### Pruebas de Hipótesis en Python

Para automatizar la validación sin necesidad de software estadístico externo, se implementó el siguiente script utilizando la suite científica de Python:

```python
import pandas as pd
import scipy.stats as stats

# 1. ANOVA de una vía para la Visita 4 (Eficacia)
visita_4 = df[df['numero_visita'] == 4]
grupos = [f['puntaje_mmse'].values for n, f in visita_4.groupby('tipo_dosis')]
f_stat, p_val_anova = stats.f_oneway(*grupos)
print(f"ANOVA Visita 4 - Estadístico: {f_stat:.4f}, p-valor: {p_val_anova:.4f}")

# 2. Prueba de Chi-cuadrado para Deserción (Seguridad)
tabla_contingencia = pd.crosstab(df['tipo_dosis'], df['estado_paciente'])
chi2, p_val_chi2, dof, expected = stats.chi2_contingency(tabla_contingencia)
print(f"Chi-cuadrado Deserción - p-valor: {p_val_chi2:.4f}")
```
## Estructura del Repositorio

El proyecto está organizado de manera modular para garantizar la escalabilidad, separando la gestión de la base de datos, el procesamiento estadístico y las consultas de extracción:

```text
├── data/
│   └── ensayo_clinico.db                # Base de datos relacional en formato SQLite
├── notebooks/
│   └── pipeline_analisis_clinico.ipynb  #Jupyter Notebook principal que contiene el flujo completo (Ingeniería de   │                                         características con SQL, manipulación de datos con Pandas y modelado       │                                         estadístico con SciPy).
└── README.md                            # Documentación principal del proyecto
```

## Cómo Ejecutar el Proyecto

Para replicar este pipeline de datos, ejecutar los análisis estadísticos o explorar el modelo de la base de datos localmente, sigue estas instrucciones:

### Prerrequisitos
Asegúrate de tener instalado Python (versión 3.8 o superior) y acceso a una terminal de comandos.

### Paso a Paso

1. **Clonar el repositorio:**
```bash
git clone https://github.com/tu-usuario/Analytics-de-Ensayos-Clinicos-Portal-de-Eficacia-y-Seguridad-Neurologica.git
cd Analytics-de-Ensayos-Clinicos-Portal-de-Eficacia-y-Seguridad-Neurologica
```
2. **Instalar las dependencias requeridas:**
Instala las librerías necesarias para el procesamiento de datos y cálculo científico ejecutando:
```bash
pip install pandas numpy scipy jupyter openpyxl
```

3. **Ejecutar el Análisis Estadístico:**
Para abrir el entorno interactivo y revisar las pruebas de hipótesis (ANOVA y Chi-cuadrado), ejecuta:
```bash
jupyter notebook notebooks/pipeline_analisis_clinico.ipynb
```
Si prefieres usar JupyterLab o la extensión de VS Code, simplemente abre el archivo directamente desde tu editor de preferencia.

4. **Explorar la Base de Datos:**
La base de datos relacional ya se encuentra compilada y lista en la ruta data/ensayo_clinico.db. Puedes conectarte a ella mediante código utilizando la librería nativa sqlite3 de Python, o de forma visual utilizando cualquier suite de gestión de bases de datos (como DB Browser for SQLite o DBeaver) para probar tus propias consultas SQL.

## Entregable Final

> **Acceso al Portal Interactivo:**
> El resultado de todo este pipeline de datos y la capa de *data storytelling* ejecutiva está publicado y disponible para su exploración pública en el siguiente enlace:
> 
> [**Ver Caso de Estudio Completo en Tableau Public**](https://public.tableau.com/app/profile/isa.v.zquez.garc.a/viz/PortaldeAnlisisdeEficaciaySeguridadenEnsayosClnicos_/Historia1)

### ¿Qué encontrará en este entregable?
Al interactuar con la **Historia de Tableau**, podrá explorar de manera interactiva:
1. **Línea Narrativa Guiada:** Un recorrido paso a paso diseñado para *stakeholders* que conecta la escala de la cohorte con los hallazgos críticos de negocio.
2. **Interactividad (UX/UI):** Implementación de *Viz-in-Tooltip* que permite auditar los efectos secundarios específicos (cefalea e insomnio) en tiempo real al posar el cursor sobre los puntos críticos de la línea temporal.
3. **Validación Visible:** Integración explícita de las conclusiones de las pruebas ANOVA y Chi-cuadrado para dar certeza estadística a cada gráfico.
