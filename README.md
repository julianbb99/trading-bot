# trading-bot
Das ist mein Trading bot für Pancake swap
cd /pfad/zum/verzeichnis/deines/codes
git init
git add .
git commit -m "Initial commit"
git remote add origin (https://github.com/julianbb99/trading-bot)
git push -u origin master
import pandas as pd
import numpy as np

# Simulierte Preisdaten erstellen (für Testzwecke)
dates = pd.date_range(start='2023-01-01', end='2023-12-31', freq='D')
vow_prices = np.random.normal(loc=0.55, scale=0.02, size=len(dates))
vusd_prices = np.random.normal(loc=0.41, scale=0.01, size=len(dates))
cake_lp_prices = np.random.normal(loc=1.02, scale=0.03, size=len(dates))

price_data = pd.DataFrame({
    'date': dates,
    'vow_price': vow_prices,
    'vusd_price': vusd_prices,
    'cake_lp_price': cake_lp_prices
})

# Startkapital und Kontostände
initial_balance_usdt = 100  # Startkapital in USDT
balance_usdt = initial_balance_usdt
balance_vow = 0
balance_vusd = 0
balance_cake_lp = 0

# Dynamischer Schwellenwert basierend auf Marktbedingungen
def dynamic_threshold(volatility, cake_lp_price, combined_price):
    if cake_lp_price > combined_price:
        return 0.10  # 10 Cent Schwelle
    else:
        return 0.05  # 5 Cent Schwelle

def calculate_volatility(price_series):
    return np.std(price_series) / np.mean(price_series)

# Verbesserte Arbitrage-Erkennung
def detect_arbitrage_opportunity_improved(vow_price, vusd_price, cake_lp_price, volatility):
    combined_price = vow_price + vusd_price
    price_difference = abs(combined_price - cake_lp_price)
    threshold = dynamic_threshold(volatility, cake_lp_price, combined_price)
    
    if price_difference > threshold:
        return combined_price > cake_lp_price
    return None

# Dynamische Anpassung des investierten Betrags basierend auf Kontostand
def dynamic_investment(balance_usdt, risk_factor=0.1):
    return min(100, balance_usdt * risk_factor)

def execute_trade_improved(arbitrage_opportunity, vow_price, vusd_price, cake_lp_price, volatility):
    global balance_usdt, balance_vow, balance_vusd, balance_cake_lp
    usdt_amount = dynamic_investment(balance_usdt)
    gas_fee = 0.1  # Simulierter Gaspreis

    if arbitrage_opportunity is True:
        # Kauf VOW und VUSD und erstelle CAKE LP
        vow_amount = usdt_amount / vow_price
        vusd_amount = usdt_amount / vusd_price
        
        if balance_usdt >= usdt_amount:
            balance_usdt -= usdt_amount
            balance_vow += vow_amount
            balance_vusd += vusd_amount
            balance_cake_lp += (vow_amount + vusd_amount)  # Vereinfachte Berechnung
            balance_usdt -= gas_fee
        
    elif arbitrage_opportunity is False:
        # Verkauf CAKE LP
        if balance_cake_lp > 0:
            balance_vow -= balance_cake_lp / 2
            balance_vusd -= balance_cake_lp / 2
            balance_usdt += (balance_vow * vow_price + balance_vusd * vusd_price)
            balance_cake_lp = 0
            balance_usdt -= gas_fee

# Durchführung des Backtests
for index, row in price_data.iterrows():
    volatility = calculate_volatility(price_data['vow_price'][:index+1])
    arbitrage_opportunity = detect_arbitrage_opportunity_improved(row['vow_price'], row['vusd_price'], row['cake_lp_price'], volatility)
    if arbitrage_opportunity is not None:
        execute_trade_improved(arbitrage_opportunity, row['vow_price'], row['vusd_price'], row['cake_lp_price'], volatility)

final_balance_usdt = balance_usdt + (balance_vow * price_data['vow_price'].iloc[-1]) + (balance_vusd * price_data['vusd_price'].iloc[-1])

print(f'Anfangskapital in USDT: {initial_balance_usdt}')
print(f'Endkapital in USDT: {final_balance_usdt}')
