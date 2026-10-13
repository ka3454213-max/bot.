import os
import time
import requests
from threading import Thread
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "MEXC Early Reversal Bot is running live!"

def run_web():
    port = int(os.environ.get("PORT", 8080))
    app.run(host='0.0.0.0', port=port)

Thread(target=run_web, daemon=True).start()

API_KEY = "mx0vglCptKlUtmb07M"
SECRET_KEY = "c19530d6d7404a509bdff4975bef5538"

SYMBOL = "BTC_USDT"
POLL_INTERVAL_SEC = 10

HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
    "Accept": "application/json, text/plain, */*",
    "Content-Type": "application/json"
}

def get_market_data():
    try:
        url = f"https://contract.mexc.com/api/v1/contract/kline/{SYMBOL}?interval=Min15&limit=60"
        response = requests.get(url, headers=HEADERS, timeout=10)
        data = response.json()
        
        if data.get("success") and data.get("data"):
            klines = data["data"]
            closes = [float(k["close"]) for k in klines]
            highs = [float(k["high"]) for k in klines]
            lows = [float(k["low"]) for k in klines]
            volumes = [float(k["volume"]) for k in klines]
            
            current_price = closes[-1]
            current_volume = volumes[-1]
            avg_volume = sum(volumes[-15:]) / 15 if len(volumes) >= 15 else current_volume
            
            gains, losses = 0, 0
            for i in range(-14, 0):
                change = closes[i] - closes[i-1]
                if change > 0:
                    gains += change
                else:
                    losses -= change
            rs = (gains / 14) / (losses / 14) if losses > 0 else 100
            rsi = 100 - (100 / (1 + rs))
            
            recent_high = max(highs[-10:-1])
            recent_low = min(lows[-10:-1])
            
            sweep_high = highs[-1] > recent_high and current_price < recent_high
            sweep_low = lows[-1] < recent_low and current_price > recent_low
            
            return {
                "price": current_price,
                "rsi": rsi,
                "volume": current_volume,
                "avg_volume": avg_volume,
                "sweep_high": sweep_high,
                "sweep_low": sweep_low
            }
    except Exception as e:
        print(f"Market Data Error: {e}", flush=True)
    return None

def trading_bot():
    print("ICT Early Reversal Bot Initialized & Scanning...", flush=True)
    
    while True:
        try:
            market = get_market_data()
            if market:
                price = market["price"]
                rsi = market["rsi"]
                vol = market["volume"]
                avg_vol = market["avg_volume"]
                sweep_high = market["sweep_high"]
                sweep_low = market["sweep_low"]
                
                print(f"[Price: ${price:.2f} | RSI: {rsi:.1f} | Vol: {vol:.0f}] Scanning Market Structure...", flush=True)
                
                is_good_volume = vol > (avg_vol * 1.1)
                
                if is_good_volume and sweep_low and (32 <= rsi <= 60):
                    print(f"EARLY REVERSAL LONG SIGNAL DETECTED at ${price:.2f} (Sweep Low + RSI Confirmed)", flush=True)
                
                elif is_good_volume and sweep_high and (40 <= rsi <= 68):
                    print(f"EARLY REVERSAL SHORT SIGNAL DETECTED at ${price:.2f} (Sweep High + RSI Confirmed)", flush=True)
                    
        except Exception as e:
            print(f"Loop Error: {e}", flush=True)
            
        time.sleep(POLL_INTERVAL_SEC)

Thread(target=trading_bot, daemon=True).start()

if __name__ == "__main__":
    port = int(os.environ.get("PORT", 8080))
    app.run(host='0.0.0.0', port=port)
