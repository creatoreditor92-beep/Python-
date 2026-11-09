# Python-
import pandas as pd
import requests
import ta
import time
import yfinance as yf

# --- 1. Telegram Settings ---
BOT_TOKEN = "APNA_BOT_TOKEN_YAHAN_DALO"  # BotFather se mila token yahan daalein
CHAT_ID = "APNA_CHAT_ID_YAHAN_DALO"      # userinfobot se mila chat ID yahan daalein

def send_telegram_message(message):
    """ Telegram par message bhejne ke liye function """
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    payload = {
        "chat_id": CHAT_ID,
        "text": message,
        "parse_mode": "Markdown"
    }
    try:
        response = requests.post(url, json=payload)
        return response.json()
    except Exception as e:
        print(f"Error sending message: {e}")

def check_signals_and_alert(df, symbol):
    # Technical Indicators calculate karein
    df['ema_50'] = ta.trend.ema_indicator(df['close'], window=50)
    df['ema_200'] = ta.trend.ema_indicator(df['close'], window=200)
    df['rsi'] = ta.momentum.rsi(df['close'], window=14)
    
    latest = df.iloc[-1]
    current_price = round(float(latest['close']), 2)
    rsi_val = round(float(latest['rsi']), 2)
    
    # Buy Signal Rule (Uptrend + RSI Oversold recovery)
    if (latest['ema_50'] > latest['ema_200']) and (latest['rsi'] < 35):
        msg = (
            f"🚨 *BUY SIGNAL ALERT* 🚨\n\n"
            f"📈 *Stock:* {symbol}\n"
            f"💰 *Price:* ₹{current_price}\n"
            f"📊 *RSI:* {rsi_val}\n"
            f"🔍 *Reason:* Uptrend active with RSI Oversold bounce."
        )
        send_telegram_message(msg)
        print("Buy Signal Sent!")
        
    # Sell Signal Rule (Downtrend + RSI Overbought zone)
    elif (latest['ema_50'] < latest['ema_200']) and (latest['rsi'] > 65):
        msg = (
            f"🔻 *SELL SIGNAL ALERT* 🔻\n\n"
            f"📉 *Stock:* {symbol}\n"
            f"💰 *Price:* ₹{current_price}\n"
            f"📊 *RSI:* {rsi_val}\n"
            f"🔍 *Reason:* Downtrend active with RSI Overbought zone."
        )
        send_telegram_message(msg)
        print("Sell Signal Sent!")
        
    else:
        print(f"[{symbol}] No Signal. RSI: {rsi_val}, Price: {current_price}")

if __name__ == "__main__":
    send_telegram_message("🤖 *Bot Started Successfully!* Monitoring market...")
    
    # Aap jis stock ya crypto ko monitor karna chahte hain uska ticker daalein (Jaise RELIANCE.NS, TCS.NS, ya BTC-USD)
    ticker = "RELIANCE.NS"
    
    while True:
        try:
            # Live market data fetch karna
            stock_data = yf.download(tickers=ticker, period="5d", interval="15m", progress=False)
            if not stock_data.empty:
                check_signals_and_alias = check_signals_and_alert(stock_data, ticker)
        except Exception as e:
            print(f"Error fetching data: {e}")
        
        # Har 5 minute (300 seconds) baad dobara check karega
        time.sleep(300)
        
