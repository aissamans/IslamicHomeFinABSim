import random
import numpy as np

# ---------------------------
# Define global simulation parameters
# ---------------------------
NUM_CLIENTS = 8000
SIMULATION_MONTHS = 36  # For example, simulate 36 months (3 years)
KHiyar_INITIAL_SCORE = 232
KHiyar_THRESHOLD = 116  # Example threshold for bank decision
NON_SALARIED_PORTIONS = [0, 0.25, 0.5, 0.75]  # Could vary in different runs
Q_PARAMETER = 0.2  # Example value; monthly income varies +/- ~20% of mean income

# ---------------------------
# Helper functions for drawing from distributions
# ---------------------------
def draw_salaried_income():
    # Salaried income distribution: 60% low, 30% medium, 10% high
    rnd = random.random()
    if rnd < 0.6:
        return random.uniform(3750 - 750, 3750 + 750)
    elif rnd < 0.9:
        return random.uniform(6250 - 1750, 6250 + 1750)
    else:
        return random.uniform(10000 - 2000, 10000 + 2000)

def draw_non_salaried_income():
    # Non-salaried income distribution: 70% low, 20% medium, 10% high
    rnd = random.random()
    if rnd < 0.7:
        mu = random.gauss(3500, 500)
    elif rnd < 0.9:
        mu = random.gauss(6000, 1000)
    else:
        mu = random.gauss(10000, 1500)
    # Add variability using Q_PARAMETER: sigma is a fraction of the mean
    sigma = Q_PARAMETER * mu / 6
    return random.gauss(mu, sigma)

def assign_house_price(income, income_class):
    # Use simple thresholds based on income class:
    if income_class == 'L':
        return random.uniform(180000, 300000)
    elif income_class == 'M':
        return random.uniform(300000, 700000)
    else:
        return random.uniform(700000, 1200000)

def calculate_rent(house_price):
    # Example rent assignment based on price ranges
    if house_price < 250000:
        return random.uniform(1000, 1300)
    elif house_price < 300000:
        return random.uniform(1300, 1600)
    elif house_price < 500000:
        return random.uniform(1600, 2000)
    elif house_price < 750000:
        return random.uniform(2000, 2700)
    elif house_price < 900000:
        return random.uniform(2700, 4000)
    else:
        return random.uniform(4000, 6000)

# ---------------------------
# Define the Client class
# ---------------------------
class Client:
    def __init__(self, id, is_salaried):
        self.id = id
        self.is_salaried = is_salaried
        
        # Initialize income based on type
        if self.is_salaried:
            self.base_income = draw_salaried_income()
            self.income_class = 'M'  # Assume salaried clients belong to middle class
        else:
            self.base_income = draw_non_salaried_income()
            # Determine income class based on mean income thresholds
            if self.base_income < 4000:
                self.income_class = 'L'
            elif self.base_income < 8000:
                self.income_class = 'M'
            else:
                self.income_class = 'H'
        
        self.age = random.randint(25, 45)  # Example: assign an initial age
        self.house_price = assign_house_price(self.base_income, self.income_class)
        self.rent = calculate_rent(self.house_price)
        
        # Monthly payment is a sum of rent (for usufruct) and redemption part
        # Here we use a placeholder calculation (this would be defined more precisely)
        self.monthly_redemption = (self.house_price * 0.01)  # Example: 1% of house price
        self.monthly_payment = self.rent + self.monthly_redemption
        
        # Account balance starts at zero or with an initial deposit if needed
        self.account_balance = 0.0
        
        # Khiyar al-Shart score and payment tracking matrix (2 rows x 12 months)
        self.khiyar_score = KHiyar_INITIAL_SCORE
        self.payment_record = np.zeros((2, 12))  # Row 0: penalty record, Row 1: binary payment status
        
        self.months_in_contract = 0  # To track progress in the contract
        self.contract_revoked = False
        self.defaulted = False
    
    def update_income(self):
        # Update monthly income with variability
        sigma = Q_PARAMETER * self.base_income / 6
        self.current_income = random.gauss(self.base_income, sigma)
    
    def decide_payment(self):
        """
        Determine payment behavior based on available income and consumption behavior.
        Here, we assume a simple model:
          - Calculate disposable income = current_income - necessary consumption.
          - If disposable income >= monthly_payment, pay in full.
          - Else, make a partial or no payment.
        """
        # Example: assume minimum consumption per month (fixed value or based on household size)
        MIN_CONSUMPTION = 12.69 * 30  # Rough monthly minimum consumption in local currency
        
        disposable_income = self.current_income - MIN_CONSUMPTION
        # For simplicity, assume client pays what they can up to the monthly_payment
        if disposable_income >= self.monthly_payment:
            payment = self.monthly_payment
            payment_status = 0  # Code "0": full payment made on time
        elif disposable_income > 0:
            payment = disposable_income
            # If only partial payment is made, determine penalty severity:
            # For instance, if only redemptions are paid, assign a lower penalty; if nothing is paid, higher penalty.
            if disposable_income >= self.monthly_redemption:
                payment_status = 1  # Code "1": covers late payments partially
            else:
                payment_status = 2  # Code "2": only minimal payment made
        else:
            payment = 0.0
            payment_status = 3  # Code "3": no payment made
        return payment, payment_status
    
    def update_payment_record(self, payment_status):
        # Update the payment record matrix for the current month (assuming month index within 0-11)
        month_idx = self.months_in_contract % 12
        # Define penalties: full payment (0 penalty), partial (e.g., 1 or 2), and no payment (3 penalty)
        penalty = {0: 0, 1: 1, 2: 2, 3: 3}[payment_status]
        self.payment_record[0, month_idx] = penalty
        # Payment status binary: 1 if payment (even if partial) was made, 0 otherwise
        self.payment_record[1, month_idx] = 1 if payment > 0 else 0
        self.khiyar_score -= penalty
    
    def check_default(self, overdue_threshold):
        # Example: if accumulated late payments exceed a threshold, mark as defaulted.
        # In practice, you would sum overdue amounts from previous months.
        # Here we use a placeholder condition.
        if self.khiyar_score < (KHiyar_INITIAL_SCORE - overdue_threshold):
            self.defaulted = True

# ---------------------------
# Define the Bank class
# ---------------------------
class Bank:
    def __init__(self, initial_capital):
        self.capital = initial_capital
        self.active_contracts = []  # List of clients with active contracts
        self.revoked_contracts = []
        self.defaulted_contracts = []
    
    def evaluate_contract(self, client):
        """
        At the end of the Khiyar al-Shart period (e.g., after 12 months),
        decide whether to continue with the contract.
        """
        if client.khiyar_score < KHiyar_THRESHOLD:
            client.contract_revoked = True
            self.revoked_contracts.append(client)
        else:
            # Contract continues; no action needed
            pass
    
    def process_payment(self, client, payment):
        # Add received payment to bank capital
        self.capital += payment

# ---------------------------
# Main Simulation Loop
# ---------------------------
def run_simulation():
    # Initialize clients; for example, a mix of salaried and non-salaried.
    clients = []
    for i in range(NUM_CLIENTS):
        # For demonstration, assign 70% salaried and 30% non-salaried; adjust as needed.
        is_salaried = random.random() < 0.7
        client = Client(id=i, is_salaried=is_salaried)
        clients.append(client)
    
    # Initialize bank with sufficient capital
    bank = Bank(initial_capital=1e9)  # Example: 1 billion in currency units
    
    # Simulation over months
    for month in range(1, SIMULATION_MONTHS + 1):
        print(f"Month {month} simulation...")
        for client in clients:
            # Skip clients whose contracts have been revoked or who have defaulted
            if client.contract_revoked or client.defaulted:
                continue

            client.months_in_contract += 1
            client.update_income()
            payment, payment_status = client.decide_payment()
            client.account_balance += client.current_income - payment  # Update account balance
            
            # Process bank payment and update client payment record only during the Khiyar period
            if client.months_in_contract <= 12:
                client.update_payment_record(payment_status)
            bank.process_payment(client, payment)
            
            # Optional: Check for default (using an example overdue threshold)
            client.check_default(overdue_threshold=6 * client.monthly_payment)
        
        # At the end of the Khiyar period (12 months), evaluate all clients
        if month == 12:
            for client in clients:
                if not client.contract_revoked and not client.defaulted:
                    bank.evaluate_contract(client)
        
        # (Optional) Record monthly statistics, update visualizations, etc.
    
    # After simulation, gather statistics
    num_revoked = len(bank.revoked_contracts)
    num_defaulted = len([c for c in clients if c.defaulted])
    print("Simulation complete.")
    print(f"Bank final capital: {bank.capital}")
    print(f"Contracts revoked: {num_revoked}")
    print(f"Contracts defaulted: {num_defaulted}")

# Run the simulation
if __name__ == "__main__":
    run_simulation()
