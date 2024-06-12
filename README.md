import sqlite3
from flask import Flask, request, render_template_string

app = Flask(__name__)

# Hardcoded sensitive information
SECRET_KEY = "supersecretkey"
DATABASE = "example.db"

# Initialize the database
def init_db():
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, username TEXT, password TEXT)")
    cursor.execute("INSERT INTO users (username, password) VALUES ('admin', 'adminpass')")
    conn.commit()
    conn.close()

# Vulnerable login function with SQL injection
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
    cursor.execute(query)
    user = cursor.fetchone()
    conn.close()
    if user:
        return "Logged in!"
    else:
        return "Invalid credentials!"

# Vulnerable form with Cross-Site Scripting (XSS)
@app.route('/xss', methods=['GET', 'POST'])
def xss():
    if request.method == 'POST':
        user_input = request.form['user_input']
        return render_template_string(f"User input: {user_input}")
    return '''
        <form method="post">
            <input type="text" name="user_input">
            <input type="submit">
        </form>
    '''

# Run the app
if __name__ == '__main__':
    init_db()
    app.run(debug=True)
