# Marriage Model
class Marriage(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    spouse1_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    spouse2_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    status = db.Column(db.String(50), default="Married")  # Married, Divorced

# Get Married
@app.route('/marry', methods=['POST'])
def marry():
    data = request.get_json()
    user1 = User.query.get(data['spouse1_id'])
    user2 = User.query.get(data['spouse2_id'])

    if user1 and user2:
        marriage = Marriage(spouse1_id=user1.id, spouse2_id=user2.id)
        db.session.add(marriage)
        db.session.commit()
        return jsonify({"message": f"{user1.username} and {user2.username} are now married!"}), 200
    return jsonify({"message": "Marriage failed."}), 400

# Inheritance System
@app.route('/inherit_wealth', methods=['POST'])
def inherit_wealth():
    data = request.get_json()
    deceased = User.query.get(data['deceased_id'])
    heir = User.query.get(data['heir_id'])

    if deceased and heir:
        heir.coins += deceased.coins
        deceased.coins = 0
        db.session.commit()
        return jsonify({"message": f"{heir.username} inherited {deceased.coins} coins."}), 200
    return jsonify({"message": "Inheritance failed."}), 400
# Government Model (Taxes & ID)
class Government(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    has_national_id = db.Column(db.Boolean, default=False)
    tax_due = db.Column(db.Integer, default=0)

# Get a National ID
@app.route('/get_national_id', methods=['POST'])
def get_national_id():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    gov_record = Government.query.filter_by(user_id=user.id).first()

    if user and user.coins >= 5000:
        user.coins -= 5000
        if not gov_record:
            gov_record = Government(user_id=user.id, has_national_id=True)
            db.session.add(gov_record)
        else:
            gov_record.has_national_id = True
        db.session.commit()
        return jsonify({"message": "National ID obtained!", "new_balance": user.coins}), 200
    return jsonify({"message": "Insufficient funds or user not found"}), 400

# Pay Taxes
@app.route('/pay_taxes', methods=['POST'])
def pay_taxes():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    gov_record = Government.query.filter_by(user_id=user.id).first()

    if user and gov_record and user.coins >= gov_record.tax_due:
        user.coins -= gov_record.tax_due
        gov_record.tax_due = 0
        db.session.commit()
        return jsonify({"message": "Taxes paid!", "new_balance": user.coins}), 200
    return jsonify({"message": "Insufficient funds or no taxes due."}), 400

# Voting System
@app.route('/vote', methods=['POST'])
def vote():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    gov_record = Government.query.filter_by(user_id=user.id).first()

    if user and gov_record and gov_record.has_national_id:
        return jsonify({"message": f"Vote cast for {data['candidate']}!"}), 200
    return jsonify({"message": "You must have a National ID to vote."}), 400
# Transport Model
class Transport(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    vehicle_type = db.Column(db.String(50), default="None")  # Bus, Taxi, Boda, Car
    fuel = db.Column(db.Integer, default=0)  # Only for personal cars

# Public Transport
@app.route('/use_transport', methods=['POST'])
def use_transport():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    transport_costs = {"bus": 200, "taxi": 500, "boda": 1000}  # Travel prices

    if user and data['mode'] in transport_costs and user.coins >= transport_costs[data['mode']]:
        user.coins -= transport_costs[data['mode']]
        db.session.commit()
        return jsonify({"message": f"Traveled using {data['mode']}.", "new_balance": user.coins}), 200
    return jsonify({"message": "Insufficient funds or invalid transport mode."}), 400

# Buy a Car
@app.route('/buy_car', methods=['POST'])
def buy_car():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    car_price = 50000  # Price of a car

    if user and user.coins >= car_price:
        user.coins -= car_price
        new_car = Transport(user_id=user.id, vehicle_type="Car", fuel=100)
        db.session.add(new_car)
        db.session.commit()
        return jsonify({"message": "Car purchased!", "new_balance": user.coins}), 200
    return jsonify({"message": "Insufficient funds or user not found"}), 400

# Refuel Car
@app.route('/refuel_car', methods=['POST'])
def refuel_car():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    car = Transport.query.filter_by(user_id=user.id, vehicle_type="Car").first()

    if user and car and user.coins >= 5000:
        user.coins -= 5000
        car.fuel = 100  # Full tank
        db.session.commit()
        return jsonify({"message": "Car refueled!", "new_balance": user.coins}), 200
    return jsonify({"message": "Insufficient funds or no car found."}), 400
# Religion Model
class Religion(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    faith = db.Column(db.String(50))  # Christianity, Islam, etc.
    reputation = db.Column(db.Integer, default=0)  # Higher reputation = better social status

# Attend Religious Service
@app.route('/attend_service', methods=['POST'])
def attend_service():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    religion = Religion.query.filter_by(user_id=user.id).first()

    if user:
        if not religion:
            religion = Religion(user_id=user.id, faith=data['faith'])
            db.session.add(religion)
        religion.reputation += 10  # Increase reputation
        db.session.commit()
        return jsonify({"message": "Service attended!", "new_reputation": religion.reputation}), 200
    return jsonify({"message": "User not found"}), 404

# Make a Donation
@app.route('/donate', methods=['POST'])
def donate_money():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    religion = Religion.query.filter_by(user_id=user.id).first()

    if user and religion and user.coins >= data['amount']:
        user.coins -= data['amount']
        religion.reputation += 20  # Donations increase reputation
        db.session.commit()
        return jsonify({"message": "Donation successful!", "new_balance": user.coins, "new_reputation": religion.reputation}), 200
    return jsonify({"message": "Insufficient funds or user not found"}), 400
# Crime Record Model
class CrimeRecord(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    crime = db.Column(db.String(100))
    fine = db.Column(db.Integer, default=0)
    jail_time = db.Column(db.Integer, default=0)  # Days in jail

# Commit a Crime (For Testing)
@app.route('/commit_crime', methods=['POST'])
def commit_crime():
    data = request.get_json()
    user = User.query.get(data['user_id'])

    if user:
        crime_type = data['crime']
        fine = data.get('fine', 500)
        jail_time = data.get('jail_time', 0)
        crime_record = CrimeRecord(user_id=user.id, crime=crime_type, fine=fine, jail_time=jail_time)
        db.session.add(crime_record)
        db.session.commit()
        return jsonify({"message": f"Committed {crime_type}. Fine: {fine}, Jail Time: {jail_time} days"}), 200
    return jsonify({"message": "User not found"}), 404

# Pay a Fine to Avoid Jail
@app.route('/pay_fine', methods=['POST'])
def pay_fine():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    crime_record = CrimeRecord.query.filter_by(user_id=user.id).first()

    if user and crime_record and user.coins >= crime_record.fine:
        user.coins -= crime_record.fine
        db.session.delete(crime_record)  # Clear crime record
        db.session.commit()
        return jsonify({"message": "Fine paid, charges cleared!", "new_balance": user.coins}), 200
    return jsonify({"message": "Insufficient funds or no crime record"}), 400

# Serve Jail Time
@app.route('/serve_jail', methods=['POST'])
def serve_jail():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    crime_record = CrimeRecord.query.filter_by(user_id=user.id).first()

    if user and crime_record:
        time_served = data['days']
        if time_served >= crime_record.jail_time:
            db.session.delete(crime_record)  # Clear record after jail time
            db.session.commit()
            return jsonify({"message": "Jail time served, record cleared!"}), 200
        return jsonify({"message": f"Still {crime_record.jail_time - time_served} days left."}), 400
    return jsonify({"message": "No jail sentence found"}), 404
# Education Model
class Education(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    level = db.Column(db.String(50), default="None")  # Primary, Secondary, University

# Enroll in School/University
@app.route('/enroll', methods=['POST'])
def enroll_school():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    education = Education.query.filter_by(user_id=user.id).first()

    if user and user.coins >= data['tuition_fee']:
        user.coins -= data['tuition_fee']
        new_level = data['level']
        if education:
            education.level = new_level
        else:
            education = Education(user_id=user.id, level=new_level)
            db.session.add(education)
        db.session.commit()
        return jsonify({"message": f"Enrolled in {new_level}!", "new_balance": user.coins}), 200
    return jsonify({"message": "Insufficient funds or user not found"}), 400

# Check Education Level
@app.route('/education_status', methods=['GET'])
def check_education():
    user_id = request.args.get('user_id')
    education = Education.query.filter_by(user_id=user_id).first()
    if education:
        return jsonify({"education_level": education.level}), 200
    return jsonify({"message": "User not found"}), 404
# Bank Account Model
class BankAccount(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    balance = db.Column(db.Integer, default=0)

# Open Bank Account
@app.route('/open_account', methods=['POST'])
def open_account():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    if user:
        new_account = BankAccount(user_id=user.id)
        db.session.add(new_account)
        db.session.commit()
        return jsonify({"message": "Bank account opened!"}), 201
    return jsonify({"message": "User not found"}), 404

# Deposit Money to Bank
@app.route('/deposit', methods=['POST'])
def deposit_money():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    account = BankAccount.query.filter_by(user_id=user.id).first()

    if user and account and user.coins >= data['amount']:
        user.coins -= data['amount']
        account.balance += data['amount']
        db.session.commit()
        return jsonify({"message": "Deposit successful!", "bank_balance": account.balance}), 200
    return jsonify({"message": "Insufficient funds or user not found"}), 400

# Withdraw Money from Bank
@app.route('/withdraw_bank', methods=['POST'])
def withdraw_bank():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    account = BankAccount.query.filter_by(user_id=user.id).first()

    if user and account and account.balance >= data['amount']:
        account.balance -= data['amount']
        user.coins += data['amount']
        db.session.commit()
        return jsonify({"message": "Withdrawal successful!", "new_balance": user.coins}), 200
    return jsonify({"message": "Insufficient bank balance or user not found"}), 400

# Apply for a Loan
@app.route('/loan', methods=['POST'])
def take_loan():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    account = BankAccount.query.filter_by(user_id=user.id).first()

    if user and account:
        loan_amount = data['amount']
        interest = int(loan_amount * 0.1)  # 10% interest
        total_repay = loan_amount + interest
        account.balance += loan_amount
        db.session.commit()
        return jsonify({"message": f"Loan approved! You owe {total_repay} coins."}), 200
    return jsonify({"message": "User not found"}), 404

# Pay Loan
@app.route('/pay_loan', methods=['POST'])
def pay_loan():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    account = BankAccount.query.filter_by(user_id=user.id).first()

    if user and account and account.balance >= data['amount']:
        account.balance -= data['amount']
        db.session.commit()
        return jsonify({"message": "Loan repayment successful!"}), 200
    return jsonify({"message": "Insufficient funds or user not found"}), 400
# Player Health Model
class PlayerHealth(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    health = db.Column(db.Integer, default=100)  # Max health = 100

# Check Player Health
@app.route('/health_status', methods=['GET'])
def health_status():
    user_id = request.args.get('user_id')
    health = PlayerHealth.query.filter_by(user_id=user_id).first()
    if health:
        return jsonify({"message": "Health checked", "health": health.health}), 200
    return jsonify({"message": "User not found"}), 404

# Go to Hospital (Restore Health)
@app.route('/hospital', methods=['POST'])
def visit_hospital():
    data = request.get_json()
    user = User.query.get(data['user_id'])
    health = PlayerHealth.query.filter_by(user_id=user.id).first()

    if user and health and user.coins >= data['treatment_cost']:
        user.coins -= data['treatment_cost']
        health.health = 100  # Restore full health
        db.session.commit()
        return jsonify({"message": "Treated successfully!", "new_balance": user.coins}), 200
    return jsonify({"message": "Insufficient funds or user not found"}), 400
