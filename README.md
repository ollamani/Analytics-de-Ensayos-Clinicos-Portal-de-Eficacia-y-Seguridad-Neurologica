# Analytics-de-Ensayos-Clinicos-Portal-de-Eficacia-y-Seguridad-Neurologica

Este proyecto implementa un pipeline de datos de extremo a extremo (End-to-End) para evaluar la eficacia y el perfil de seguridad de un fármaco neurológico en una cohorte simulada de **1,000 pacientes** a lo largo de **4 visitas cronológicas**. 

**Link al Dashboard Interactivo:**
(https://public.tableau.com/app/profile/isa.v.zquez.garc.a/viz/PortaldeAnlisisdeEficaciaySeguridadenEnsayosClnicos_/Historia1)

---

## Arquitectura del Proyecto

El proyecto se divide en tres capas de desarrollo:
1. **Almacenamiento (SQL):** Estructuración y persistencia de los datos clínicos mediante SQLite.
2. **Procesamiento y Validación Estadística (Python):** Limpieza, agregación y ejecución de pruebas de hipótesis analíticas (`SciPy`, `Pandas`).
3. **Visualización (Tableau):** Capa ejecutiva con arquitectura *Viz-in-Tooltip* e Historias narrativas.

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

```python
df_anova = pd.read_sql_query(query_anova, conexion)
# Separar los puntajes en arreglos independientes para cada grupo
placebo = df_anova[df_anova['Dosis'] == 'Placebo']['Puntaje_Final']
dosis_baja = df_anova[df_anova['Dosis'] == 'Dosis Baja (5mg)']['Puntaje_Final']
dosis_alta = df_anova[df_anova['Dosis'] == 'Dosis Alta (10mg)']['Puntaje_Final']
f_estadistico, p_valor = stats.f_oneway(placebo, dosis_baja, dosis_alta)
```
   
2. **Prueba de Seguridad (Chi-cuadrado):** Se analizó la tasa de deserción por grupo. 
   Con un $p = 0.02$, se rechaza la hipótesis nula, demostrando que la deserción en la Dosis Alta 
   está vinculada a la intolerancia al tratamiento (cefalea/insomnio).

```python
df_chi = pd.read_sql_query(query_chi, conexion)
tabla_contingencia = pd.crosstab(df_chi['nombre_dosis'], df_chi['estado_paciente'])
chi2, p_valor, dof, esperados = stats.chi2_contingency(tabla_contingencia)
```
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

Para explorar los scripts completos y las consultas de base de datos detalladas, el proyecto está organizado de la siguiente manera:

/sql/: Contiene los scripts de creación de tablas (schema.sql) y las consultas analíticas de extracción.

/notebooks/: Contiene el Jupyter Notebook con el análisis exploratorio de datos (EDA), la limpieza con Pandas y las pruebas completas de SciPy.

/data/: Base de datos relacional en formato SQLite (ensayo_clinico.db).
