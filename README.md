from flask import Flask, render_template, request, redirect, url_for, session, flash
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash

app = Flask(__name__)
app.secret_key = 'clave_secreta_super_segura'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///usuarios.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

# Modelo de Usuario
class Usuario(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    nombre_usuario = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(100), unique=True, nullable=False)
    password = db.Column(db.String(200), nullable=False)

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        nombre_usuario = request.form['nombre_usuario']
        email = request.form['email']
        password = generate_password_hash(request.form['password'])

        nuevo_usuario = Usuario(nombre_usuario=nombre_usuario, email=email, password=password)

        try:
            db.session.add(nuevo_usuario)
            db.session.commit()
            flash('Registro exitoso. Ahora puedes iniciar sesión.')
            return redirect(url_for('login'))
        except:
            flash('Error: el correo ya está registrado.')
            return redirect(url_for('register'))

    return render_template('register.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        email = request.form['email']
        password = request.form['password']

        usuario = Usuario.query.filter_by(email=email).first()

        if usuario and check_password_hash(usuario.password, password):
            session['usuario_id'] = usuario.id
            session['nombre_usuario'] = usuario.nombre_usuario
            return redirect(url_for('dashboard'))
        else:
            flash('Email o contraseña incorrectos.')
            return redirect(url_for('login'))

    return render_template('login.html')

@app.route('/dashboard')
def dashboard():
    if 'usuario_id' in session:
        return render_template('dashboard.html', nombre=session['nombre_usuario'])
    else:
        return redirect(url_for('login'))

@app.route('/logout')
def logout():
    session.clear()
    flash('Sesión cerrada.')
    return redirect(url_for('login'))

if __name__ == '__main__':
    with app.app_context():
        db.create_all()
    app.run(debug=True)
    
