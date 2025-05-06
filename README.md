import datetime
import json

class RedeemSystem:
    def __init__(self):
        # Predefined list of valid codes and their associated rewards
        self.valid_codes = {
            "WELCOME2025": "10% Discount",
            "SUMMERFUN": "Free Shipping",
            "EXTRADEAL": "Buy 1 Get 1 Free"
        }
        self.redeemed_codes = {}  # Tracks redeemed codes

    def redeem_code(self, code):
        current_time = datetime.datetime.now()
        if code in self.redeemed_codes:
            return f"Code '{code}' has already been redeemed on {self.redeemed_codes[code]}."

        if code in self.valid_codes:
            self.redeemed_codes[code] = current_time.strftime("%Y-%m-%d %H:%M:%S")
            reward = self.valid_codes[code]
            return f"Code '{code}' redeemed successfully! Reward: {reward}"
        else:
            return f"Invalid code: '{code}'. Please try again."

    def log_redeemed_codes(self):
        # Save redeemed codes to a JSON file
        with open("redeemed_codes_log.json", "w") as file:
            json.dump(self.redeemed_codes, file, indent=4)
        print("Redeemed codes logged successfully.")

if __name__ == "__main__":
    redeem_system = RedeemSystem()
    print("Welcome to the Redeem Code System!")

    while True:
        user_input = input("\nEnter a redeem code (or type 'exit' to quit): ").strip()
        if user_input.lower() == "exit":
            break

        response = redeem_system.redeem_code(user_input)
        print(response)

    # Log redeemed codes on exit
    redeem_system.log_redeemed_codes()
    print("Thank you for using the Redeem Code System!")
