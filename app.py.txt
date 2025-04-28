from flask import Flask, render_template, request, redirect, send_file
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas
import os
import sqlite3
from datetime import datetime

app = Flask(__name__)
DB_NAME = 'remisiones.db'

# Crear la base de datos si no existe
def init_db():
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    c.execute('''
        CREATE TABLE IF NOT EXISTS remisiones (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            fecha TEXT,
            cliente TEXT,
            direccion TEXT,
            ciudad TEXT,
            telefono TEXT,
            conductor TEXT,
            cedula TEXT,
            placa TEXT,
            ruta TEXT,
            producto TEXT,
            cantidad INTEGER,
            factura TEXT,
            observaciones TEXT
        )
    ''')
    conn.commit()
    conn.close()

init_db()

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        # Tomar los datos del formulario
        fecha = request.form['fecha']
        cliente = request.form['cliente']
        direccion = request.form['direccion']
        ciudad = request.form['ciudad']
        telefono = request.form['telefono']
        conductor = request.form['conductor']
        cedula = request.form['cedula']
        placa = request.form['placa']
        ruta = request.form['ruta']
        producto = request.form['producto']
        cantidad = request.form['cantidad']
        factura = request.form['factura']
        observaciones = request.form['observaciones']

        # Guardar en la base de datos
        conn = sqlite3.connect(DB_NAME)
        c = conn.cursor()
        c.execute('''
            INSERT INTO remisiones (fecha, cliente, direccion, ciudad, telefono, conductor, cedula, placa, ruta, producto, cantidad, factura, observaciones)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        ''', (fecha, cliente, direccion, ciudad, telefono, conductor, cedula, placa, ruta, producto, cantidad, factura, observaciones))
        conn.commit()
        last_id = c.lastrowid
        conn.close()

        # Generar PDF
        pdf_path = f"remision_{last_id}.pdf"
        generar_pdf(pdf_path, last_id, fecha, cliente, direccion, ciudad, telefono, conductor, cedula, placa, ruta, producto, cantidad, factura, observaciones)
        
        return send_file(pdf_path, as_attachment=True)

    # Mostrar historial
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    c.execute('SELECT * FROM remisiones')
    remisiones = c.fetchall()
    conn.close()
    return render_template('index.html', remisiones=remisiones)

def generar_pdf(filename, id_remision, fecha, cliente, direccion, ciudad, telefono, conductor, cedula, placa, ruta, producto, cantidad, factura, observaciones):
    c = canvas.Canvas(filename, pagesize=letter)
    width, height = letter

    # Logo
    if os.path.exists("static/logo.jpg"):
        c.drawImage("static/logo.jpg", 40, height - 80, width=150, preserveAspectRatio=True)

    # Texto
    c.setFont("Helvetica-Bold", 14)
    c.drawString(200, height - 50, f"Remisión N° {id_remision}")

    c.setFont("Helvetica", 10)
    c.drawString(40, height - 120, f"Fecha: {fecha}")
    c.drawString(40, height - 140, f"Cliente: {cliente}")
    c.drawString(40, height - 160, f"Dirección: {direccion}")
    c.drawString(40, height - 180, f"Ciudad: {ciudad}")
    c.drawString(40, height - 200, f"Teléfono: {telefono}")
    c.drawString(40, height - 220, f"Conductor: {conductor}")
    c.drawString(40, height - 240, f"Cédula: {cedula}")
    c.drawString(40, height - 260, f"Placa: {placa}")
    c.drawString(40, height - 280, f"Ruta: {ruta}")
    c.drawString(40, height - 300, f"Producto: {producto}")
    c.drawString(40, height - 320, f"Cantidad: {cantidad}")
    c.drawString(40, height - 340, f"Número de Factura: {factura}")
    c.drawString(40, height - 360, f"Observaciones: {observaciones}")

    c.save()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
