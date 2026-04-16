# Datos_Educativos_ARG_PYTHON
!pip install pyeph
import pyeph
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
from  matplotlib.ticker import FuncFormatter
import pandas as pd
import numpy as np
import pyeph as eph # Importamos la librería pyeph

# --- 1. CONFIGURACIÓN INICIAL ---
# Definimos el año y trimestre que queremos analizar.
año_analisis = 2019
trimestre_analisis = 2 # ¡Segundo trimestre de 2019!

# Mapeo de códigos de región a nombres (según la documentación de la EPH)
MAPEO_REGION = {
    1: 'Gran Buenos Aires',
    40: 'Noroeste',
    41: 'Noreste',
    42: 'Cuyo',
    43: 'Pampeana',
    44: 'Patagonia'
}

# --- 2. CARGAR DATOS USANDO PYEPH ---
print(f"Cargando datos de EPH para {año_analisis}T{trimestre_analisis} usando pyeph...")

# Inicializar df_eph_hogar a None.
df_eph_hogar = None

try:

    # El parámetro para el tipo de base es 'base_type'
    df_eph_hogar = eph.get(data="eph", year=año_analisis, period=trimestre_analisis, base_type='hogar')
    print(f"Datos de Hogares para {año_analisis}T{trimestre_analisis} cargados exitosamente con pyeph.")
except Exception as e:
    print(f"Error al cargar los datos con pyeph para {año_analisis}T{trimestre_analisis}: {e}")
    print("Por favor, verifica lo siguiente:")
    print("  1. Que tengas conexión a internet.")
    print("  2. Que el paquete 'pyeph' esté correctamente instalado y actualizado (puedes intentar '!pip install pyeph --upgrade' de nuevo).")
    print("  3. Que el año y trimestre que intentas descargar estén disponibles a través de pyeph y uses los parámetros correctos (year, period, base_type).")


# --- COMPROBACIÓN DESPUÉS DE LA CARGA ---
# Solo procedemos si el DataFrame se cargó correctamente.
if df_eph_hogar is None:
    print("\n¡ATENCIÓN! No se pudo cargar el DataFrame de la EPH. El análisis no puede continuar.")
    exit() # Ahora sí, salimos si el DataFrame es None.

# Esta línea solo se ejecutará si df_eph_hogar NO es None
print(f"\nRegistros totales de hogares cargados: {len(df_eph_hogar)}")

# --- 3. PREPROCESAMIENTO DE VARIABLES ---
# Convertir columnas a tipo numérico y manejar posibles valores "No sabe/No responde" (NaN).
df_eph_hogar['IPCF'] = pd.to_numeric(df_eph_hogar['IPCF'], errors='coerce')
df_eph_hogar['DECCFR'] = pd.to_numeric(df_eph_hogar['DECCFR'], errors='coerce') # Decil de IPCF
df_eph_hogar['REGION'] = pd.to_numeric(df_eph_hogar['REGION'], errors='coerce')
df_eph_hogar['PONDIH'] = pd.to_numeric(df_eph_hogar['PONDIH'], errors='coerce')

# Eliminar filas con NaN en las columnas clave para el análisis.
df_eph_filtered = df_eph_hogar.dropna(subset=['IPCF', 'DECCFR', 'REGION', 'PONDIH'])

# Mapear códigos numéricos de región a nombres descriptivos.
df_eph_filtered['REGION_DESC'] = df_eph_filtered['REGION'].map(MAPEO_REGION)

# Volver a filtrar si alguna región no fue mapeada o si REGION era NaN.
df_eph_filtered = df_eph_filtered.dropna(subset=['REGION_DESC'])

print(f"\nRegistros válidos para análisis de nivel socioeconómico por región: {len(df_eph_filtered)}")

# --- 4. ANÁLISIS DEL NIVEL SOCIOECONÓMICO POR REGIÓN ---

print("\n--- Análisis de Ingreso Per Cápita Familiar (IPCF) por Región ---")

# Calcular el IPCF promedio ponderado por región.
ipcf_promedio_ponderado_por_region = df_eph_filtered.groupby('REGION_DESC').apply(
    lambda x: (x['IPCF'] * x['PONDIH']).sum() / x['PONDIH'].sum()
)

print("\nIPCF Promedio Ponderado por Región:")
print(ipcf_promedio_ponderado_por_region.sort_values(ascending=False))

print("\n--- Distribución de Deciles de Ingreso Per Cápita Familiar (DECCFR) por Región ---")

# Calcular la distribución de deciles de IPCF por región (ponderado).
distribucion_deciles_por_region = df_eph_filtered.pivot_table(
    index='REGION_DESC',
    columns='DECCFR',
    values='PONDIH',
    aggfunc='sum',
    fill_value=0
)

# Convertir la suma de ponderadores a porcentajes para cada fila.
distribucion_deciles_por_region_porcentaje = distribucion_deciles_por_region.apply(
    lambda x: (x / x.sum()) * 100, axis=1
)

print("\nDistribución Porcentual de Deciles de IPCF por Región (Ponderado):")
print(distribucion_deciles_por_region_porcentaje.round(2))
import matplotlib.pyplot as plt

plt.figure(figsize=(12, 7))
ipcf_promedio_ponderado_por_region.plot(kind='bar', color='skyblue')
plt.title(f'IPCF Promedio Ponderado por Región ({año_analisis} T{trimestre_analisis})')
plt.ylabel('Ingreso Per Cápita Familiar (IPCF)')
plt.xlabel('Región')
plt.xticks(rotation=45, ha='right')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()

plt.figure(figsize=(14, 8))
distribucion_deciles_por_region_porcentaje.plot(kind='bar', stacked=True, cmap='viridis')
plt.title(f'Distribución de Deciles de IPCF por Región (Ponderado) ({año_analisis} T{trimestre_analisis})')
plt.ylabel('Porcentaje de Hogares (%)')
plt.xlabel('Región')
plt.xticks(rotation=45, ha='right')
plt.legend(title='Decil IPCF', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()
