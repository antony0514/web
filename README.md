pip install flask
from flask import Flask, render_template, request, redirect
import sqlite3

app = Flask(__name__)

def init_db():
    with sqlite3.connect("inventario.db") as conn:
        conn.execute("""
        CREATE TABLE IF NOT EXISTS productos (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            nombre TEXT NOT NULL,
            tienda TEXT NOT NULL,
            cantidad INTEGER NOT NULL
        )
        """)

@app.route('/')
def index():
    conn = sqlite3.connect("inventario.db")
    productos = conn.execute("SELECT nombre, tienda, cantidad FROM productos").fetchall()
    conn.close()
    return render_template("index.html", productos=productos)

@app.route('/agregar', methods=['POST'])
def agregar():
    nombre = request.form['nombre']
    tienda = request.form['tienda']
    cantidad = int(request.form['cantidad'])

    conn = sqlite3.connect("inventario.db")
    cursor = conn.cursor()
    cursor.execute("SELECT id, cantidad FROM productos WHERE nombre=? AND tienda=?", (nombre, tienda))
    producto = cursor.fetchone()

    if producto:
        nueva_cantidad = producto[1] + cantidad
        cursor.execute("UPDATE productos SET cantidad=? WHERE id=?", (nueva_cantidad, producto[0]))
    else:
        cursor.execute("INSERT INTO productos (nombre, tienda, cantidad) VALUES (?, ?, ?)",
                       (nombre, tienda, cantidad))

    conn.commit()
    conn.close()
    return redirect('/')

@app.route('/movimiento', methods=['POST'])
def movimiento():
    nombre = request.form['mov_nombre']
    tipo = request.form['tipo']
    cantidad = int(request.form['mov_cantidad'])

    conn = sqlite3.connect("inventario.db")
    cursor = conn.cursor()
    cursor.execute("SELECT id, cantidad FROM productos WHERE nombre=?", (nombre,))
    producto = cursor.fetchone()

    if producto:
        nueva_cantidad = producto[1] + cantidad if tipo == 'entrada' else max(producto[1] - cantidad, 0)
        cursor.execute("UPDATE productos SET cantidad=? WHERE id=?", (nueva_cantidad, producto[0]))
        conn.commit()

    conn.close()
    return redirect('/')

if __name__ == '__main__':
    init_db()
    app.run(debug=True)
    <!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Inventario de Tiendas</title>
</head>
<body>
    <h1>Gestión de Inventario</h1>

    <h2>Agregar Producto</h2>
    <form action="/agregar" method="post">
        <input type="text" name="nombre" placeholder="Nombre del producto" required>
        <input type="text" name="tienda" placeholder="Tienda" required>
        <input type="number" name="cantidad" placeholder="Cantidad" required>
        <button type="submit">Agregar</button>
    </form>

    <h2>Registrar Entrada o Salida</h2>
    <form action="/movimiento" method="post">
        <input type="text" name="mov_nombre" placeholder="Nombre del producto" required>
        <select name="tipo">
            <option value="entrada">Entrada</option>
            <option value="salida">Salida</option>
        </select>
        <input type="number" name="mov_cantidad" placeholder="Cantidad" required>
        <button type="submit">Registrar</button>
    </form>

    <h2>Inventario Actual</h2>
    <table border="1">
        <tr>
            <th>Producto</th>
            <th>Tienda</th>
            <th>Cantidad</th>
        </tr>
        {% for p in productos %}
        <tr>
            <td>{{ p[0] }}</td>
            <td>{{ p[1] }}</td>
            <td>{{ p[2] }}</td>
        </tr>
        {% endfor %}
    </table>
</body>
</html>
python app.py
