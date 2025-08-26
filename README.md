# Ejercicio-Tesla-Gamestop-IBM
# Ejercicio de prueba
# tesla_gamestop_analysis.py (versión corregida con Matplotlib)
import yfinance as yf
import pandas as pd
import requests
from bs4 import BeautifulSoup
import matplotlib.pyplot as plt
import warnings

# Configuración inicial
warnings.filterwarnings("ignore")
plt.style.use('ggplot')

print("=== ANÁLISIS DE TESLA Y GAMESTOP ===")

# Función make_graph con Matplotlib (corregida)
def make_graph(stock_data, revenue_data, stock):
    """
    Función para graficar precio de acciones y revenue con Matplotlib
    """
    try:
        # Crear figura con dos subplots
        fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(12, 8))
        fig.suptitle(f'{stock} - Historical Share Price and Revenue', fontsize=16)
        
        # Gráfico 1: Precio de acciones
        stock_data['Date'] = pd.to_datetime(stock_data['Date'])
        stock_data_filtered = stock_data[stock_data['Date'] <= '2021-06-30']
        
        ax1.plot(stock_data_filtered['Date'], stock_data_filtered['Close'], 
                color='blue', linewidth=2, label='Share Price')
        ax1.set_ylabel('Price ($US)', fontsize=12)
        ax1.set_title('Historical Share Price')
        ax1.grid(True, alpha=0.3)
        ax1.legend()
        
        # Gráfico 2: Revenue
        revenue_data['Date'] = pd.to_datetime(revenue_data['Date'])
        revenue_data_filtered = revenue_data[revenue_data['Date'] <= '2021-06-30']
        
        ax2.plot(revenue_data_filtered['Date'], revenue_data_filtered['Revenue'].astype(float), 
                color='green', linewidth=2, label='Revenue')
        ax2.set_ylabel('Revenue ($US Millions)', fontsize=12)
        ax2.set_xlabel('Date', fontsize=12)
        ax2.set_title('Historical Revenue')
        ax2.grid(True, alpha=0.3)
        ax2.legend()
        
        # Formatear fechas
        for ax in [ax1, ax2]:
            plt.sca(ax)
            plt.xticks(rotation=45)
        
        plt.tight_layout()
        plt.show()
        
    except Exception as e:
        print(f"Error al crear gráfico: {e}")
        
# QUESTION 1: EXTRACT TESLA STOCK DATA
print("\n1. EXTRACTING TESLA STOCK DATA...")

# Crear ticker object para Tesla
tesla = yf.Ticker("TSLA")

# Extraer datos históricos
tesla_data = tesla.history(period="max")

# Resetear índice
tesla_data.reset_index(inplace=True)

# Mostrar primeras 5 filas
print("Primeras 5 filas de Tesla Data:")
print(tesla_data.head())

# QUESTION 2: EXTRACT TESLA REVENUE DATA (WEB SCRAPING)
print("\n2. EXTRACTING TESLA REVENUE DATA...")

# Descargar página web
url_tesla = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0220EN-SkillsNetwork/labs/project/revenue.htm"
html_data_tesla = requests.get(url_tesla).text

# Parsear HTML
soup_tesla = BeautifulSoup(html_data_tesla, 'html.parser')

# Crear DataFrame vacío para Tesla revenue
tesla_revenue = pd.DataFrame(columns=['Date', 'Revenue'])

# Encontrar la tabla correcta (índice 1 para quarterly revenue)
tables = soup_tesla.find_all('tbody')
if len(tables) > 1:
    table_body = tables[1]
    
    # Extraer datos de la tabla
    for row in table_body.find_all('tr'):
        cols = row.find_all('td')
        if len(cols) >= 2:
            date = cols[0].text.strip()
            revenue = cols[1].text.strip()
            
            new_row = pd.DataFrame({
                'Date': [date],
                'Revenue': [revenue]
            })
            tesla_revenue = pd.concat([tesla_revenue, new_row], ignore_index=True)

# Limpiar datos de revenue
tesla_revenue['Revenue'] = tesla_revenue['Revenue'].str.replace(',', '').str.replace('$', '')
tesla_revenue = tesla_revenue[tesla_revenue['Revenue'] != '']
tesla_revenue.dropna(inplace=True)
tesla_revenue['Revenue'] = tesla_revenue['Revenue'].astype(float)

# Mostrar últimas 5 filas
print("Últimas 5 filas de Tesla Revenue:")
print(tesla_revenue.tail())

# QUESTION 3: EXTRACT GAMESTOP STOCK DATA
print("\n3. EXTRACTING GAMESTOP STOCK DATA...")

# Crear ticker object para GameStop
gme = yf.Ticker("GME")

# Extraer datos históricos
gme_data = gme.history(period="max")

# Resetear índice
gme_data.reset_index(inplace=True)

# Mostrar primeras 5 filas
print("Primeras 5 filas de GameStop Data:")
print(gme_data.head())

# QUESTION 4: EXTRACT GAMESTOP REVENUE DATA (WEB SCRAPING)
print("\n4. EXTRACTING GAMESTOP REVENUE DATA...")

# Descargar página web
url_gme = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0220EN-SkillsNetwork/labs/project/stock.html"
html_data_gme = requests.get(url_gme).text

# Parsear HTML
soup_gme = BeautifulSoup(html_data_gme, 'html.parser')

# Crear DataFrame vacío para GameStop revenue
gme_revenue = pd.DataFrame(columns=['Date', 'Revenue'])

# Encontrar la tabla correcta
tables_gme = soup_gme.find_all('tbody')
if len(tables_gme) > 1:
    table_body_gme = tables_gme[1]
    
    # Extraer datos de la tabla
    for row in table_body_gme.find_all('tr'):
        cols = row.find_all('td')
        if len(cols) >= 2:
            date = cols[0].text.strip()
            revenue = cols[1].text.strip()
            
            new_row = pd.DataFrame({
                'Date': [date],
                'Revenue': [revenue]
            })
            gme_revenue = pd.concat([gme_revenue, new_row], ignore_index=True)

# Limpiar datos de revenue
gme_revenue['Revenue'] = gme_revenue['Revenue'].str.replace(',', '').str.replace('$', '')
gme_revenue = gme_revenue[gme_revenue['Revenue'] != '']
gme_revenue.dropna(inplace=True)
gme_revenue['Revenue'] = gme_revenue['Revenue'].astype(float)

# Mostrar últimas 5 filas
print("Últimas 5 filas de GameStop Revenue:")
print(gme_revenue.tail())

# QUESTION 5: PLOT TESLA STOCK GRAPH
print("\n5. PLOTTING TESLA GRAPH...")
make_graph(tesla_data, tesla_revenue, 'Tesla')

# QUESTION 6: PLOT GAMESTOP STOCK GRAPH
print("\n6. PLOTTING GAMESTOP GRAPH...")
make_graph(gme_data, gme_revenue, 'GameStop')

print("\n=== ANÁLISIS COMPLETADO ===")

# Información adicional
print(f"\nRESUMEN:")
print(f"Tesla Data: {tesla_data.shape}")
print(f"Tesla Revenue: {tesla_revenue.shape}")
print(f"GameStop Data: {gme_data.shape}")
print(f"GameStop Revenue: {gme_revenue.shape}")
