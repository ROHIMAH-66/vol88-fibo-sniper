
import os
import requests
import json
from datetime import datetime, timezone

BOT_TOKEN = os.getenv("BOT_TOKEN")
CHAT_ID = os.getenv("CHAT_ID")

# Data candle OHLC dimasukkan manual melalui candles.json
def send_telegram(message):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    requests.post(url, json={
        "chat_id": CHAT_ID,
        "text": message
    }, timeout=20)

def analyze(candles):
    if len(candles) < 10:
        return "WAIT: Data candle belum cukup."

    highs = [float(c["high"]) for c in candles]
    lows = [float(c["low"]) for c in candles]
    closes = [float(c["close"]) for c in candles]
    opens = [float(c["open"]) for c in candles]

    # Fractal 2-2 pada candle yang sudah terkonfirmasi
    swing_highs = []
    swing_lows = []

    for i in range(2, len(candles) - 2):
        if all(highs[i] > highs[j]
               for j in range(i-2, i+3) if j != i):
            swing_highs.append((i, highs[i]))

        if all(lows[i] < lows[j]
               for j in range(i-2, i+3) if j != i):
            swing_lows.append((i, lows[i]))

    if not swing_highs or not swing_lows:
        return "NO TRADE: Swing belum valid."

    # Cari swing terakhir yang berurutan
    candidates = []
    for hi_i, hi in swing_highs:
        for lo_i, lo in swing_lows:
            if hi_i > lo_i:
                candidates.append(("BUY", lo_i, lo_i, hi_i, lo, hi))
            elif lo_i > hi_i:
                candidates.append(("SELL", hi_i, hi_i, lo_i, hi, lo))

    if not candidates:
        return "NO TRADE: Urutan swing belum valid."

    direction, _, start_i, end_i, start, end = max(
        candidates, key=lambda x: x[4] if False else x[3]
    )

    # Pastikan swing berasal dari rangkaian candle yang cukup baru
    if end_i < len(candles) - 10:
        return "NO TRADE: Swing terlalu lama."

    size = abs(end - start)
    if size == 0:
        return "NO TRADE: Jarak swing nol."

    # Momentum candle terakhir
    bodies = [abs(closes[i] - opens[i]) for i in range(max(0, len(candles)-11), len(candles)-1)]
    avg_body = sum(bodies) / len(bodies) if bodies else 0
    last_body = abs(closes[-1] - opens[-1])

    if avg_body and last_body < 1.5 * avg_body:
        return "WAIT: Momentum belum memenuhi syarat."

    if direction == "BUY":
        levels = {
            "0.500": end - size * 0.500,
            "0.618": end - size * 0.618,
            "0.705": end - size * 0.705
        }
        instruction = "Tarik Fibo dari Swing Low ke Swing High."
    else:
        levels = {
            "0.500": end + size * 0.500,
            "0.618": end + size * 0.618,
            "0.705": end + size * 0.705
        }
        instruction = "Tarik Fibo dari Swing High ke Swing Low."

    return (
        f"VOL 88 FIBO SNIPER\n"
        f"TIMEFRAME: M1\n"
        f"SETUP: {direction}\n"
        f"Swing awal: {start}\n"
        f"Swing akhir: {end}\n\n"
        f"FIBONACCI:\n"
        + "\n".join(f"{k}: {v:.3f}" for k, v in levels.items())
        + f"\n\n{instruction}\n"
        "Tunggu konfirmasi candle sebelum entry."
    )

if __name__ == "__main__":
    with open("candles.json", "r") as f:
        candles = json.load(f)

    result = analyze(candles)
    print(result)

    if BOT_TOKEN and CHAT_ID:
        send_telegram(result)
  
