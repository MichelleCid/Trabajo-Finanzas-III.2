# Trabajo-Finanzas-III.2
Trabajo Finanzas III, Google Colab
# Instalación necesaria 
!pip install fpdf --quiet
import math
from scipy.stats import norm
from google.colab import files
from fpdf import FPDF
import matplotlib.pyplot as plt

# Función para agregar texto al PDF con salto de línea adecuado
def agregar_texto(pdf, texto):
    pdf.set_xy(10, pdf.get_y() + 6)
    pdf.multi_cell(0, 8, texto)

# 1) VALORIZACIÓN DE BONOS
def valor_bono(cupon, tasa_interes, años, valor_nominal):
    t = range(1, años + 1)
    valor_presente_cupones = sum(cupon / (1 + tasa_interes) ** i for i in t)
    valor_presente_nominal = valor_nominal / (1 + tasa_interes) ** años
    valor = valor_presente_cupones + valor_presente_nominal
    return valor

# Parámetros bono
cupon = 50
tasa_interes = 0.05
años = 15
valor_nominal = 300
bono_valor = valor_bono(cupon, tasa_interes, años, valor_nominal)

# 2) VALORIZACIÓN DE ACCIONES (Modelo Gordon)
def valor_accion(dividendo, tasa_crecimiento, tasa_descuento):
    if tasa_descuento <= tasa_crecimiento:
        raise ValueError("La tasa de descuento debe ser mayor que la tasa de crecimiento.")
    valor = dividendo / (tasa_descuento - tasa_crecimiento)
    return valor

# Parámetros acción
dividendo = 80
tasa_crecimiento = 0.03
tasa_descuento = 0.07
accion_valor = valor_accion(dividendo, tasa_crecimiento, tasa_descuento)

# 3) VALORIZACIÓN DE FUTUROS
def valor_futuro(valor_presente, tasa_interes, tiempo):
    valor = valor_presente * (1 + tasa_interes) ** tiempo
    return valor

# Parámetros futuro
valor_presente = 250
tasa_interes_futuro = 0.35
tiempo = 1
futuro_valor = valor_futuro(valor_presente, tasa_interes_futuro, tiempo)

# 4) VALORIZACIÓN DE FORWARD
def calcular_forward(S, r_d, r_f, T_dias):
    # Tasa diaria base y extranjera (asumiendo simple para días)
    F = S * (1 + r_d * T_dias / 360) / (1 + r_f * T_dias / 360)
    return F

def valorar_forward(F, S_f, N):
    V = (F - S_f) * N
    return V

# Parámetros forward
S = 520
r_d = 0.007   # 0.7% anual
r_f = 0.03    # 3% anual
T_forward = 180  # días
N = 1000
F_calc = calcular_forward(S, r_d, r_f, T_forward)

S_f = 1.2200  # Spot actual para valoración
forward_valor = valorar_forward(F_calc, S_f, N)

# 5) VALORIZACIÓN DE OPCIONES (Modelo Black-Scholes Call)
def valor_opcion_call(S, K, T, r, sigma):
    d1 = (math.log(S / K) + (r + 0.5 * sigma ** 2) * T) / (sigma * math.sqrt(T))
    d2 = d1 - sigma * math.sqrt(T)
    C = S * norm.cdf(d1) - K * math.exp(-r * T) * norm.cdf(d2)
    return C

# Parámetros opción call
S_call = 100
K = 120
T_call = 1
r_call = 0.05
sigma = 0.2
call_valor = valor_opcion_call(S_call, K, T_call, r_call, sigma)

# 6) VALORIZACIÓN DE SWAP (Tasa fija vs tasa variable)
notional = 1_000_000
fixed_rate = 0.04
floating_rate = 0.05
n = 1  # años
swap_valor = notional * (floating_rate - fixed_rate) * n

# 7) VALORIZACIÓN DE FRA (Forward Rate Agreement) 3x6
r1 = 0.04  # tasa 3 meses
r2 = 0.05  # tasa 6 meses
fra_valor = ((1 + r2*0.5) / (1 + r1*0.25) - 1) / 0.25

# Mostrar resultados en consola
print("Resultados obtenidos:\n")
print(f"1) Valor del bono: ${bono_valor:.2f}")
print(f"2) Valor de la acción: ${accion_valor:.2f}")
print(f"3) Valor del futuro: ${futuro_valor:.2f}")
print(f"4.1) Tipo de cambio forward: {F_calc:.4f}")
print(f"4.2) Valor del contrato forward: ${forward_valor:.2f}")
print(f"5) Valor de la opción call: ${call_valor:.2f}")
print(f"6) Valor del Swap (Tasa flotante - fija): ${swap_valor:.2f}")
print(f"7) Valor Tasa FRA 3x6 estimada: {fra_valor*100:.2f}%")

# Gráfico de barras con los valores (se muestra y se guarda para PDF)
nombres = ["Bono", "Acción", "Futuro", "Forward", "Opción Call", "Swap", "FRA"]
valores = [bono_valor, accion_valor, futuro_valor, forward_valor, call_valor, swap_valor, fra_valor]

plt.figure(figsize=(10, 6))
plt.bar(nombres, valores, color='skyblue')
plt.title("Valoración de Activos Financieros")
plt.ylabel("Valor ($)")
plt.xticks(rotation=45)
plt.grid(axis='y')
plt.tight_layout()
plt.savefig("grafico_valores.png")
plt.show()

# Crear archivo PDF con resultados y gráfico
pdf = FPDF()
pdf.add_page()

# Portada
pdf.set_font("Arial", "B", 22)
pdf.cell(0, 20, "Informe de Valoración de Activos Financieros", ln=True, align="C")
pdf.ln(20)

pdf.set_font("Arial", size=14)
pdf.cell(0, 10, "Integrantes: Michelle Cid, Martina Marchant y Bianca Antican.", ln=True, align="C")
pdf.cell(0, 10, "Curso: Finanzas III", ln=True, align="C")
pdf.cell(0, 10, "Profesor: Carlos Cavieres.", ln=True, align="C")
pdf.cell(0, 10, "Ayudante: Valentina Aguilera.", ln=True, align="C")
pdf.cell(0, 10, "Fecha: Junio 2025.", ln=True, align="C")
pdf.ln(20)
pdf.set_font("Arial", "I", 12)
pdf.cell(0, 10, "Universidad de Santiago de Chile", ln=True, align="C")

# Página de resultados
pdf.add_page()
pdf.set_font("Arial", "B", 16)
pdf.cell(0, 10, "Resumen de Resultados", ln=True, align='C')
pdf.ln(10)
pdf.set_font("Arial", size=12)

# Agregar resultados
agregar_texto(pdf, f"1) Valor del bono: ${bono_valor:.2f}")
agregar_texto(pdf, f"2) Valor de la acción: ${accion_valor:.2f}")
agregar_texto(pdf, f"3) Valor del futuro: ${futuro_valor:.2f}")
agregar_texto(pdf, f"4.1) Tipo de cambio forward: {F_calc:.4f}")
agregar_texto(pdf, f"4.2) Valor del contrato forward: ${forward_valor:.2f}")
agregar_texto(pdf, f"5) Valor de la opción call: ${call_valor:.2f}")
agregar_texto(pdf, f"6) Valor del Swap (Tasa flotante - fija): ${swap_valor:.2f}")
agregar_texto(pdf, f"7) Valor Tasa FRA 3x6 estimada: {fra_valor*100:.2f}%")

pdf.ln(10)

# Insertar gráfico guardado
pdf.set_font("Arial", "B", 14)
pdf.cell(0, 10, "Gráfico de Valoración de Activos Financieros", ln=True, align="C")
pdf.image("grafico_valores.png", x=20, w=170)

pdf.ln(15)

# Conclusión
pdf.set_font("Arial", "B", 14)
pdf.cell(0, 10, "Conclusión", ln=True)
pdf.set_font("Arial", size=12)
conclusion_text = (
    "Los resultados obtenidos muestran la valoración financiera de distintos instrumentos "
    "clave en los mercados financieros. Cada activo presenta características y riesgos "
    "diferentes, reflejados en sus respectivos valores calculados. Estos cálculos permiten "
    "a los inversionistas tomar decisiones informadas sobre la compra, venta o mantenimiento "
    "de dichos activos, considerando factores como tasas de interés, volatilidad y tiempo. "
    "El análisis visual mediante el gráfico facilita la comparación rápida de los valores "
    "relativos entre los instrumentos evaluados."
)
agregar_texto(pdf, conclusion_text)

# Guardar y descargar PDF
nombre_archivo = "Informe_Valoracion_Activos_Financieros.pdf"
pdf.output(nombre_archivo)
files.download(nombre_archivo)
