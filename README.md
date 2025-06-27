import logging
from binance.client import Client
from binance.enums import *
from binance.exceptions import BinanceAPIException
import sys

# Configure Logging
logging.basicConfig(
    filename='trading_bot.log',
    filemode='a',
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

class BasicBot:
    def __init__(self, api_key, api_secret, testnet=True):
        self.client = Client(api_key, api_secret)
        if testnet:
            self.client.FUTURES_URL = 'https://testnet.binancefuture.com/fapi'

        logging.info("Initialized Binance Futures client.")

    def place_order(self, symbol, side, order_type, quantity, price=None):
        try:
            logging.info(f"Placing {order_type} {side} order for {symbol}, quantity: {quantity}, price: {price}")

            if order_type == ORDER_TYPE_MARKET:
                order = self.client.futures_create_order(
                    symbol=symbol,
                    side=side,
                    type=order_type,
                    quantity=quantity
                )
            elif order_type == ORDER_TYPE_LIMIT:
                order = self.client.futures_create_order(
                    symbol=symbol,
                    side=side,
                    type=order_type,
                    timeInForce=TIME_IN_FORCE_GTC,
                    quantity=quantity,
                    price=price
                )
            else:
                raise ValueError("Unsupported order type.")

            logging.info(f"Order placed: {order}")
            return order
        except BinanceAPIException as e:
            logging.error(f"API Error: {e.message}")
            print(f"Binance API error: {e.message}")
        except Exception as e:
            logging.error(f"General Error: {str(e)}")
            print(f"Error placing order: {e}")
        return None

def validate_symbol(symbol):
    return symbol.endswith("USDT")

def validate_float(val):
    try:
        return float(val)
    except:
        raise ValueError("Invalid number")

if __name__ == "__main__":
    # Credentials
    api_key = input("Enter your Binance Testnet API Key: ").strip()
    api_secret = input("Enter your Binance Testnet Secret Key: ").strip()

    bot = BasicBot(api_key, api_secret)

    print("\nSupported order types: MARKET, LIMIT")
    symbol = input("Enter trading pair (e.g., BTCUSDT): ").strip().upper()
    if not validate_symbol(symbol):
        sys.exit("Only USDT-margined pairs are supported.")

    side = input("Enter order side (BUY or SELL): ").strip().upper()
    if side not in ['BUY', 'SELL']:
        sys.exit("Invalid side entered.")

    order_type = input("Enter order type (MARKET or LIMIT): ").strip().upper()
    if order_type not in ['MARKET', 'LIMIT']:
        sys.exit("Invalid order type entered.")

    quantity = validate_float(input("Enter quantity: "))
    price = None

    if order_type == 'LIMIT':
        price = validate_float(input("Enter limit price: "))

    order = bot.place_order(
        symbol=symbol,
        side=SIDE_BUY if side == 'BUY' else SIDE_SELL,
        order_type=order_type,
        quantity=quantity,
        price=price
    )

    if order:
        print("Order placed successfully!")
        print(order)
    else:
        print("Failed to place order.")
 
