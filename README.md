[Proyecto1.py](https://github.com/user-attachments/files/28246528/Proyecto1.py)
import tkinter as tk
from tkinter import messagebox, ttk
from sistema_reservas import SistemaReservas
from excepciones import ClienteError, ServicioError, ReservaError
from servicios import ReservaSala, AlquilerEquipo, AsesoriaEspecializada

# ------------------ SISTEMA GLOBAL ------------------
sistema = SistemaReservas()

# ------------------ FUNCIONES ------------------
def registrar_cliente():
    nombre = entry_nombre.get().strip()
    documento = entry_documento.get().strip()
    correo = entry_correo.get().strip()
    telefono = entry_telefono.get().strip()
    
    try:
        cliente = sistema.registrar_cliente(nombre, documento, correo, telefono)
        messagebox.showinfo("Éxito", f"{cliente} registrado exitosamente")
        limpiar_campos_cliente()
    except ClienteError as e:
        messagebox.showerror("Error", str(e))

def registrar_servicio():
    tipo = tipo_servicio_var.get()
    nombre = entry_nombre_servicio.get().strip()
    valor_text = entry_valor_servicio.get().strip()
    extra_text = entry_extra.get().strip()

    try:
        if not nombre:
            raise ServicioError("El nombre o tema del servicio es obligatorio")
        if not valor_text:
            raise ServicioError("El valor del servicio es obligatorio")
        valor = float(valor_text)

        if tipo == "Sala":
            if not extra_text:
                raise ServicioError("La capacidad es obligatoria para salas")
            capacidad = int(extra_text)
            servicio = sistema.registrar_servicio_sala(nombre, capacidad, valor)
        elif tipo == "Equipo":
            if not extra_text:
                raise ServicioError("La marca es obligatoria para equipos")
            servicio = sistema.registrar_servicio_equipo(nombre, extra_text, valor)
        else:
            if not extra_text:
                raise ServicioError("El especialista es obligatorio para asesorías")
            servicio = sistema.registrar_servicio_asesoria(nombre, extra_text, valor)

        messagebox.showinfo("Éxito", f"{servicio.descripcion()} registrado")
        limpiar_campos_servicio()
    except (ServicioError, ValueError) as e:
        messagebox.showerror("Error", str(e))

def obtener_tipo_servicio_objeto(servicio):
    if isinstance(servicio, ReservaSala):
        return "Sala"
    if isinstance(servicio, AlquilerEquipo):
        return "Equipo"
    if isinstance(servicio, AsesoriaEspecializada):
        return "Asesoria"
    return ""


def crear_reserva():
    doc_cliente = entry_doc_reserva.get().strip()
    tipo_servicio = tipo_servicio_reserva_var.get()
    nombre_servicio = entry_servicio_reserva.get().strip()
    fecha = entry_fecha.get().strip()
    cantidad_text = entry_cantidad.get().strip()

    try:
        if not nombre_servicio:
            raise ReservaError("El nombre del servicio es obligatorio")
        if not cantidad_text:
            raise ReservaError("La cantidad es obligatoria")
        cantidad = float(cantidad_text)

        cliente = None
        for c in sistema.clientes:
            if c.documento == doc_cliente:
                cliente = c
                break
        
        if not cliente:
            messagebox.showerror("Error", "Cliente no encontrado")
            return
        
        servicio = None
        servicio_tipo_equivocado = None
        for s in sistema.servicios:
            if tipo_servicio == "Sala" and isinstance(s, ReservaSala):
                if s.nombre_sala == nombre_servicio:
                    servicio = s
                    break
            elif tipo_servicio == "Equipo" and isinstance(s, AlquilerEquipo):
                if s.tipo_equipo == nombre_servicio:
                    servicio = s
                    break
            elif tipo_servicio == "Asesoria" and isinstance(s, AsesoriaEspecializada):
                if s.tema == nombre_servicio:
                    servicio = s
                    break
            # Detect same name in a different category
            if not servicio:
                if isinstance(s, ReservaSala) and s.nombre_sala == nombre_servicio:
                    servicio_tipo_equivocado = "Sala"
                elif isinstance(s, AlquilerEquipo) and s.tipo_equipo == nombre_servicio:
                    servicio_tipo_equivocado = "Equipo"
                elif isinstance(s, AsesoriaEspecializada) and s.tema == nombre_servicio:
                    servicio_tipo_equivocado = "Asesoria"
        
        if not servicio:
            if servicio_tipo_equivocado:
                messagebox.showerror("Error", f"Servicio encontrado como tipo {servicio_tipo_equivocado}. Seleccione el tipo de servicio correcto.")
            else:
                messagebox.showerror("Error", "Servicio no encontrado")
            return
        
        reserva = sistema.crear_reserva(cliente, servicio, fecha, cantidad)
        messagebox.showinfo("Éxito", f"{reserva}")
        limpiar_campos_reserva()
    except (ReservaError, ValueError) as e:
        messagebox.showerror("Error", str(e))

def listar_reservas():
    reservas = sistema.listar_reservas_activas()
    if not reservas:
        messagebox.showinfo("Información", "No hay reservas activas")
        return
    
    texto = "RESERVAS ACTIVAS:\n\n"
    for r in reservas:
        texto += f"{r}\n\n"
    
    messagebox.showinfo("Reservas", texto)

def limpiar_campos_cliente():
    entry_nombre.delete(0, tk.END)
    entry_documento.delete(0, tk.END)
    entry_correo.delete(0, tk.END)
    entry_telefono.delete(0, tk.END)

def limpiar_campos_servicio():
    entry_nombre_servicio.delete(0, tk.END)
    entry_valor_servicio.delete(0, tk.END)
    entry_extra.delete(0, tk.END)

def limpiar_campos_reserva():
    entry_doc_reserva.delete(0, tk.END)
    entry_servicio_reserva.delete(0, tk.END)
    entry_fecha.delete(0, tk.END)
    entry_cantidad.delete(0, tk.END)

# ------------------ INTERFAZ ------------------
root = tk.Tk()
root.title("Software FJ - Sistema de Reservas")
root.geometry("500x600")

# Notebook para pestañas
notebook = ttk.Notebook(root)
notebook.pack(fill='both', expand=True, padx=10, pady=10)

# Pestaña Clientes
frame_clientes = ttk.Frame(notebook)
notebook.add(frame_clientes, text="Clientes")

ttk.Label(frame_clientes, text="Nombre").pack(pady=5)
entry_nombre = ttk.Entry(frame_clientes)
entry_nombre.pack()

ttk.Label(frame_clientes, text="Documento").pack(pady=5)
entry_documento = ttk.Entry(frame_clientes)
entry_documento.pack()

ttk.Label(frame_clientes, text="Correo").pack(pady=5)
entry_correo = ttk.Entry(frame_clientes)
entry_correo.pack()

ttk.Label(frame_clientes, text="Teléfono").pack(pady=5)
entry_telefono = ttk.Entry(frame_clientes)
entry_telefono.pack()

ttk.Button(frame_clientes, text="Registrar Cliente", command=registrar_cliente).pack(pady=10)

# Pestaña Servicios
frame_servicios = ttk.Frame(notebook)
notebook.add(frame_servicios, text="Servicios")

ttk.Label(frame_servicios, text="Tipo de Servicio").pack(pady=5)
tipo_servicio_var = tk.StringVar(value="Sala")
ttk.OptionMenu(frame_servicios, tipo_servicio_var, "Sala", "Sala", "Equipo", "Asesoria").pack()

ttk.Label(frame_servicios, text="Nombre/Tema").pack(pady=5)
entry_nombre_servicio = ttk.Entry(frame_servicios)
entry_nombre_servicio.pack()

ttk.Label(frame_servicios, text="Valor (Hora/Día)").pack(pady=5)
entry_valor_servicio = ttk.Entry(frame_servicios)
entry_valor_servicio.pack()

extra_label = ttk.Label(frame_servicios, text="Capacidad")
extra_label.pack(pady=5)
entry_extra = ttk.Entry(frame_servicios)
entry_extra.pack()

def actualizar_label_extra(*args):
    tipo = tipo_servicio_var.get()
    if tipo == "Sala":
        extra_label.config(text="Capacidad")
    elif tipo == "Equipo":
        extra_label.config(text="Marca")
    else:
        extra_label.config(text="Especialista")

tipo_servicio_var.trace_add("write", actualizar_label_extra)

ttk.Button(frame_servicios, text="Registrar Servicio", command=registrar_servicio).pack(pady=10)

# Pestaña Reservas
frame_reservas = ttk.Frame(notebook)
notebook.add(frame_reservas, text="Reservas")

ttk.Label(frame_reservas, text="Documento Cliente").pack(pady=5)
entry_doc_reserva = ttk.Entry(frame_reservas)
entry_doc_reserva.pack()

ttk.Label(frame_reservas, text="Tipo Servicio").pack(pady=5)
tipo_servicio_reserva_var = tk.StringVar(value="Sala")
ttk.OptionMenu(frame_reservas, tipo_servicio_reserva_var, "Sala", "Sala", "Equipo", "Asesoria").pack()

ttk.Label(frame_reservas, text="Nombre Servicio (Sala/Equipo/Tema)").pack(pady=5)
entry_servicio_reserva = ttk.Entry(frame_reservas)
entry_servicio_reserva.pack()

ttk.Label(frame_reservas, text="Fecha").pack(pady=5)
entry_fecha = ttk.Entry(frame_reservas)
entry_fecha.pack()

ttk.Label(frame_reservas, text="Cantidad (Horas/Días)").pack(pady=5)
entry_cantidad = ttk.Entry(frame_reservas)
entry_cantidad.pack()

ttk.Button(frame_reservas, text="Crear Reserva", command=crear_reserva).pack(pady=10)
ttk.Button(frame_reservas, text="Listar Reservas Activas", command=listar_reservas).pack(pady=5)

root.mainloop()
