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
