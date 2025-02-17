from flask import Flask, render_template, request, redirect, url_for, flash
import mysql.connector
from flask_login import LoginManager, UserMixin, login_user, login_required, logout_user

# App configuration
app = Flask(__name__)
app.config.from_object('config.Config')

# Initialize MySQL connection
db = mysql.connector.connect(
    host="local host",
    user="root",
    password="admin123",
    database="patient_records"
)
cursor = db.cursor()

# Flask-Login Configuration
login_manager = LoginManager()
login_manager.init_app(app)
login_manager.login_view = "login"

# User class for Flask-Login
class User(UserMixin):
    def __init__(self, id, username, email, role):
        self.id = id
        self.username = username
        self.email = email
        self.role = role

# Load user
@login_manager.user_loader
def load_user(user_id):
    cursor.execute("SELECT * FROM Users WHERE user_id = %s", (user_id,))
    user = cursor.fetchone()
    if user:
        return User(user[0], user[1], user[2], user[3])
    return None

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        email = request.form['email']
        password = request.form['password']

        cursor.execute("SELECT * FROM Users WHERE email = %s AND password = %s", (email, password))
        user = cursor.fetchone()

        if user:
            user_obj = User(user[0], user[1], user[2], user[3])
            login_user(user_obj)
            return redirect(url_for('dashboard'))

        flash('Invalid credentials!', 'danger')
    
    return render_template('login.html')

@app.route('/dashboard')
@login_required
def dashboard():
    return render_template('dashboard.html')

if __name__ == '__main__':
    app.run(debug=True)
