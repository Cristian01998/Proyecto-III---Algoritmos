PROYECTO III
----
## DOCUMENTACION EXTERNA

Este documento proporciona información detallada sobre el uso, características y su funcionalidades. Nuestra aplicación se llama: Monaco Editor, la cual cumple con las funciones de un editor de texto. 
----
    import tkinter as tk # el módulo principal que proporciona las clases y funciones necesarias para crear interfaces gráficas 
    from tkinter import filedialog, messagebox, simpledialog # Importa submódulos específicos,  proporciona funciones y clases útiles para realizar tareas
    import subprocess  # para abrir el archivo PDF

    class TextEditorApp:
        """Clase que representa la aplicación de editor de texto con GUI."""
        def __init__(self, root):
            # Inicialización de la aplicación
            self.root = root #está almacenando la referencia a la ventana principal en una variable interna de la instancia de la clase.
            self.root.title("Monaco Editor") #  establecer el título de la ventana principal de la aplicación

            # Configuración inicial
            self.setup_gui()

            # Inicialización de variables
            self.current_file = None #determina el estado actual del editor en relación con los archivos
            self.font_family = "Arial"
            self.font_size = 12
            self.theme = "Predeterminado"

            # Configuración de eventos y acciones de cierre
            self.bind_events() #contribuye a la funcionalidad interactiva y al comportamiento del editor en respuesta a las acciones del usuario.
            self.root.protocol("WM_DELETE_WINDOW", self.on_closing) # brinda la posibilidad de realizar acciones específicas antes de que la aplicación se cierre.

            # Configuración de la apariencia del área de texto y de números de línea
            self.text_area.configure(font=(self.font_family, self.font_size)) #asegura que el texto dentro del área de texto tenga una fuente específica y un tamaño específico
            self.line_number_area.configure(font=(self.font_family, self.font_size)) # asegura que los números de línea en el área correspondiente tengan una fuente específica y un tamaño específico

            # Actualización inicial de los números de línea
            self.update_line_numbers() #  es un método utilizado para mantener sincronizados los números de línea en el área de números de línea con el contenido actual del área de texto.

        def setup_gui(self): 
            """Configura la interfaz gráfica de la aplicación."""
            # Configuración de la interfaz gráfica,  mejora la modularidad y la legibilidad del código, se divide en funciones más pequeñas y específicas, lo que facilita la comprensión y el mantenimiento del código.
            self.create_widgets() #  se encarga de organizar y estructurar la creación de los componentes de la interfaz gráfica

        def create_widgets(self):
            """Crea los elementos de la interfaz gráfica."""
            #  estas líneas organizan la creación de los elementos esenciales de la interfaz  de Monaco Editor: como el área de números de línea, las barras de desplazamiento, la barra de estado, el área de texto principal y el menú principal. 
            self.create_line_number_area()
            self.create_scrollbars()
            self.create_status_bar()
            self.create_text_area()
            self.create_menu()

        def create_line_number_area(self):
            """Crea el área de números de línea."""
            #  creación y configuración del área de números de línea en la área principal de texto del editor.
            self.line_number_area = tk.Text(self.root, width=4, wrap="none", takefocus=0) # Se utiliza como el widget principal ,  Establece el ancho del área de números de línea.
            self.line_number_area.pack(side=tk.LEFT, fill=tk.Y) #organiza y posiciona el área de números de línea en el lado izquierdo de la ventana principal, y hace que se expanda verticalmente.
            self.line_number_area.config(state=tk.DISABLED) #  el área de números de línea no puede ser editada por el usuario.

        def create_scrollbars(self):
            # Creación de la horientacion de las barras de desplazamiento
            """Crea las barras de desplazamiento."""
            self.scrollbar_y = tk.Scrollbar(self.root, orient=tk.VERTICAL) # desplazamiento vertical
            self.scrollbar_y.pack(side=tk.RIGHT, fill=tk.Y) # organiza y posiciona la barra de desplazamiento vertical en el lado derecho de la ventana principal
            self.scrollbar_x = tk.Scrollbar(self.root, orient=tk.HORIZONTAL) # desplazamiento horizontal
            self.scrollbar_x.pack(side=tk.BOTTOM, fill=tk.X) #  se utiliza para empaquetar y mostrar la barra de desplazamiento horizontal en la interfaz gráfica.

        def create_status_bar(self):
            """Crea la barra de estado."""
            self.status_bar = tk.Label(self.root, text="Línea: 1 | Columna: 0", bd=1, relief=tk.SUNKEN, anchor=tk.W) # Configura el estilo del relieve de la barra de estado
            self.status_bar.pack(side=tk.BOTTOM, fill=tk.X) #  se expanda horizontalmente para ocupar todo el espacio horizontal disponible.

        def create_text_area(self):
            """Crea el área de texto."""
            self.text_area = tk.Text(self.root, wrap="none", yscrollcommand=self.scrollbar_y.set, xscrollcommand=self.scrollbar_x.set, undo=True, autoseparators=True, maxundo=-1) # : Asocia la barra de desplazamiento vertical y horizontal, con el área de texto para permitir el desplazamiento,  Habilita la funcionalidad de deshacer en el área de texto.. 
            self.text_area.pack(expand="yes", fill="both") # permite que el área de texto se expanda tanto vertical como horizontalmente para ocupar todo el espacio disponible. 
            # Configuración de eventos relacionados con el área de texto
            self.text_area.bind("<Motion>", self.update_status_bar) # activa la función cada vez que hay un evento de movimiento del ratón en el área de texto
            self.text_area.bind("<KeyRelease>", self.update_status_bar) # activa la funcion cada vez que se libera una tecla en el área de texto.
            self.text_area.bind("<Configure>", self.update_scrollbars)  # activa la funcion cada vez que hay un evento de configuración en el área de texto.
            self.text_area.bind("<MouseWheel>", self.on_mousewheel) #  activa la funcion cada vez que se utiliza la rueda del ratón en el área de texto.
            self.text_area.bind("<Button-1>", self.resaltar_linea_action) # resalta la línea en la que se hizo clic. 

            # Configuración de las barras de desplazamiento
            self.scrollbar_y.config(command=self.text_area.yview) # Utiliza el método config para establecer el comando de la barra de desplazamiento vertical.
            self.scrollbar_x.config(command=self.text_area.xview) # Utiliza el método config para establecer el comando de la barra de desplazamiento horizontal.

            # Configuración de eventos adicionales relacionados con el área de texto (funcion Rehacer)
            self.text_area.bind("<Configure>", self.update_line_numbers) # Este enlace activa la función cada vez que hay un evento de configuración en el área de texto.
            self.text_area.bind("<Control-r>", self.rehacer_action) # cada vez que se presiona la combinación de teclas "Control + r" o "Control + R".
            self.text_area.bind("<Control-R>", self.rehacer_action)

        def create_menu(self):
            # Creación del menú principal
            """Crea el menú principal."""
            self.menu_bar = tk.Menu(self.root) # Este objeto cumple la funcion como el contenedor para los diferentes menús y opciones del menú.
            self.root.config(menu=self.menu_bar) # establece  como el menú principal de la ventana principal.

            # Definición de los elementos del menú y sus submenús
            menus = {
                "Archivo": ["Abrir", "Guardar", "Guardar como", "-", "Salir"],
                "Editar": ["Deshacer/ Ctrl Z", "Rehacer/ Ctrl R", "-", "Copiar/ Ctrl C", "Pegar/ Ctrl V"],
                "Configuración": ["Cambiar tema", "Cambiar tamaño de letra", "Cambiar tipo de letra"],
                "Ayuda": ["Información", "Manual de Usuario", "Desarrollado por"]
            }

            # Creación de los elementos del menú
            # automatiza la creación y adición de menús a la barra de menú principal utilizando la información proporcionada en el diccionario menus.
            for menu_name, menu_items in menus.items():
                self.create_menu_items(menu_name, menu_items)

        def create_menu_items(self, menu_name, menu_items): # se encarga de crear y agregar elementos a un menú específico dentro de la barra de menú principal.
            menu = tk.Menu(self.menu_bar, tearoff=0) #  representa un menú específico.
            self.menu_bar.add_cascade(label=menu_name, menu=menu) #  esta función toma el nombre de un menú y la lista de elementos asociados a ese menú.

            for item in menu_items:
                if item == "-":
                    # Agregar separador en el menú
                    menu.add_separator()
                elif item == "Información":
                    # Agregar comando para la opción "Información"
                    menu.add_command(label=item, command=self.informacion_action)
                elif item == "Manual de Usuario":
                    # Agregar comando para la opción "Manual de Usuario"
                    menu.add_command(label=item, command=self.abrir_manual_usuario)
                elif item == "Desarrollado por":
                    # Agregar comando para la opción "Desarrollado por"
                    menu.add_command(label=item, command=self.Desarrollado_por_action)
                else:
                    # Verificar si es "tipo de letra" o "tamaño de letra"
                    if "tipo de letra" in item:
                        # Agregar comando para cambiar el tipo de letra
                        menu.add_command(label=item, command=self.cambiar_tipo_letra_action, accelerator="")
                    elif "tamaño de letra" in item:
                        # Agregar comando para cambiar el tamaño de letra
                        menu.add_command(label=item, command=self.cambiar_tamano_letra_action, accelerator="")
                    else:
                        # Obtener el nombre de la función y el acelerador
                        command_name = f"{item.lower().replace(' ', '_')}_action"
                        accelerator = getattr(self, f"{item.lower().replace(' ', '_')}_accelerator", "")
                    
                        # Agregar comando al menú y configurar acelerador si existe
                        menu.add_command(label=item, command=getattr(self, command_name, lambda: None), accelerator=accelerator)
                    if accelerator:
                        self.root.bind(accelerator, getattr(self, command_name, lambda x: None))

        def abrir_manual_usuario(self):
            """Abre el manual de usuario en un archivo PDF."""
            # proporciona una manera de abrir el manual de usuario en formato PDF y maneja la posibilidad de que el archivo no esté disponible
            try:
                subprocess.Popen(["C:\Program Files\Adobe\Acrobat DC\Acrobat\Acrobat.exe", r"C:\Users\Cristian\Desktop\Curso ALGORITMOS 2023\Agoritmos en C++\Proyecto III\Manual de Usuario.pdf"])
            except FileNotFoundError:
                messagebox.showerror("Error", "No se pudo encontrar el archivo de PDF predeterminado. Por favor, abre el manual de usuario manualmente.")

        def abrir_action(self):
            """Acción de abrir archivo."""
            # esta función facilita al usuario la selección de un archivo de texto mediante un cuadro de diálogo y proporciona la ruta del archivo.
            print("Abriendo archivo")
            file_path = filedialog.askopenfilename(filetypes=[("Archivos de texto", "*.txt"), ("Todos los archivos", "*.*")])

            if file_path:
                # Abre el archivo seleccionado en modo lectura ("r"), Lee el contenido del archivo y lo almacena en la variable content.
                with open(file_path, "r") as file: 
                    content = file.read()
                    self.text_area.delete("1.0", tk.END)
                    self.text_area.insert(tk.END, content)

                self.current_file = file_path # carga el contenido del archivo seleccionado en el área de texto del editor, garantiza que el área de texto muestre el contenido recién abierto.
                self.update_line_numbers()

        def guardar_action(self):
            # Acción de guardar archivo
            print("Guardando archivo")
            if self.current_file: # Verifica si hay un archivo actualmente abierto
                with open(self.current_file, "w") as file: # Abre el archivo actual en modo escritura ("w").
                    content = self.text_area.get("1.0", tk.END) 
                    file.write(content)
            else:
                self.guardar_como_action()
                # gestiona la acción de guardar, ya sea escribiendo en el archivo actual si existe o llamando a la función para guardar como si no hay un archivo

        def guardar_como_action(self):
            # Acción de guardar archivo como
            print("Guardando archivo como")
            file_path = filedialog.asksaveasfilename(defaultextension=".txt", filetypes=[("Archivos de texto", "*.txt;*.cpp;*.cs;*.py"), ("Todos los archivos", "*.*")])

            if file_path: # Verifica si el usuario seleccionó una ruta y un nombre de archivo antes de cerrar el cuadro de diálogo.
                with open(file_path, "w") as file:
                    content = self.text_area.get("1.0", tk.END)
                    file.write(content)

                self.current_file = file_path # Asigna la nueva ruta del archivo a la variable. 
                self.update_line_numbers() # actualiza los números de línea en el área de la columna de numeros.

        def copiar_action(self, event=None):
            # Acción de copiar texto
            print("Copiando texto")
            self.text_area.event_generate("<<Copy>>") #  está generando un evento de copiac en el área de texto

        def pegar_action(self, event=None):
            # Acción de pegar texto
            print("Pegando texto")
            self.text_area.event_generate("<<Paste>>") #  está generando un evento de pegar en el área de texto

        def deshacer_action(self, event=None):
            # Acción de deshacer
            print("Deshaciendo")
            try:
                if self.text_area.edit_modified(): # Verifica si hay cambios no guardados en el área de texto.
                    self.text_area.edit_undo() #  deshacer el último cambio.
                    self.update_line_numbers() # actualizar los números de línea después de deshacer los cambios.
            except tk.TclError: # Captura y omite cualquier error que pueda ocurrir durante el intento de deshacer los cambios.
                pass

        def rehacer_action(self, event=None):
            # Acción de rehacer
            print("Rehaciendo")
            try:
                self.text_area.edit_redo() # rehace el ultimo cambio 
                self.update_line_numbers()  # actualizar los números de línea después de rehacer los cambios.
            except tk.TclError: #  # Captura y omite cualquier error que pueda ocurrir durante el intento de rehacer los cambios.
                pass

        def update_status_bar(self, event=None):
            # Actualización de la barra de estado
            cursor_position = self.text_area.index(tk.CURRENT) # obtener la posición actual del cursor
            line, column = cursor_position.split('.') # Divide la posición del cursor en dos partes: la línea y la columna. La posición del cursor está en el formato "línea.columna".
            status_text = f"Línea: {line} | Columna: {column}" # Crea un texto informativo que muestra la línea y la columna actuales del cursor.
            self.status_bar.config(text=status_text) # actualiza dinámicamente la barra de estado de la interfaz gráfica para reflejar la posición actual del cursor.

        def update_scrollbars(self, event=None):
            # Actualización de las barras de desplazamiento
            self.scrollbar_y.set(0.0, tk.END) # establece la posición inicial de la barra de desplazamiento vertical desde la parte superior hasta el final del contenido en el eje Y.
            self.scrollbar_x.set(0.0, tk.END) # establece la posición inicial de la barra de desplazamiento horizontal desde la izquierda hasta el final del contenido en el eje X.

        def update_line_numbers(self, event=None):
            # Actualización de los números de línea
            self.line_number_area.config(state=tk.NORMAL) # Configura el área de números de línea para que sea editable.
            self.line_number_area.delete("1.0", tk.END) # Elimina cualquier número de línea existente en el área de números de línea.
            num_lines = str(self.text_area.index(tk.END).split('.')[0]) #  para obtener el número total de líneas en el área de texto. Luego, convierte este número a una cadena.
            self.line_number_area.insert(tk.END, '\n'.join(str(i) for i in range(1, int(num_lines) + 1))) # Inserta nuevos números de línea en el área de números de línea. Utiliza una comprensión de lista para generar una secuencia de números.
            self.line_number_area.config(state=tk.DISABLED) # Configura el área de números de línea como no editable después de la actualización.
            # Calcula la cantidad de dígitos en el número total de líneas y ajusta el ancho del área de números de línea en consecuencia.
            last_line_num_digits = len(str(num_lines))
            self.line_number_area.config(width=last_line_num_digits)

        def on_mousewheel(self, event): #  responde al evento de rueda del ratón
            # Este método permite desplazar verticalmente tanto el área de números de línea como el área de texto cuando el usuario utiliza la rueda del ratón.
            self.line_number_area.yview_scroll(int(-1 * event.delta), "units")
            self.text_area.yview_scroll(int(-1 * event.delta), "units")

        def focus_in_event(self, event): # garantiza que los números de línea se actualicen asegurando que siempre se muestre la información más reciente sobre las líneas del área de texto.
            # Evento de enfoque
            self.update_line_numbers() #  para actualizar los números de línea en el área

        def bind_events(self):
            # Vinculación de eventos
            self.text_area.bind("<FocusIn>", self.focus_in_event) # cuando el usuario hace clic, se ejecutará el método
            self.text_area.bind("<FocusOut>", lambda event: None) # evento de desenfoque
    #
        def on_closing(self):
            # Acción al cerrar la aplicación
            if self.is_text_edited(): # verificar si ha habido cambios en el texto del área de texto desde la última vez que se guardó.
                save_option = messagebox.askyesnocancel("Cerrar", "¿Desea guardar los cambios antes de cerrar?") # Muestra un cuadro de diálogo que pregunta al usuario si desea guardar los cambios
                if save_option is not None: 
                    # Si el usuario elige "Sí" para guardar los cambios, se llama al método guardar_action para guardar el contenido del área de texto. En cualquier caso (si elige guardar o no), se procede a cerrar la aplicación 
                    if save_option:
                        self.guardar_action()
                    self.root.destroy() 
            else:
                self.root.destroy() # Si no ha habido cambios en el texto (según la verificación inicial), la aplicación se cierra directamente sin preguntar al usuario.

        def is_text_edited(self): # Verificación de cambios en el texto
            # proporciona una forma conveniente de verificar si el contenido del área de texto ha sido modificado
            return self.text_area.edit_modified()

        def resaltar_linea_action(self, event):
            # Acción de resaltar línea
            cursor_position = self.text_area.index(tk.CURRENT)
            line_number = cursor_position.split('.')[0]
            start_index = f"{line_number}.0"
            end_index = f"{line_number}.end"

            self.text_area.tag_remove("highlight", "1.0", tk.END)
            self.line_number_area.tag_remove("highlight", "1.0", tk.END)

            self.text_area.tag_add("highlight", start_index, end_index)
            self.line_number_area.tag_add("highlight", start_index, end_index)

            self.text_area.tag_configure("highlight", background="yellow")
            self.line_number_area.tag_configure("highlight", background="yellow")

        def cambiar_tema_action(self):
            # Acción de cambiar tema
            theme_options = ["Predeterminado", "Oscuro", "Claro"]
            options_string = "\n".join(f"{i + 1}. {theme}" for i, theme in enumerate(theme_options))
            prompt = f"Seleccione el tema:\n{options_string}"
            selected_index = simpledialog.askinteger("Cambiar Tema", prompt, minvalue=1, maxvalue=len(theme_options), initialvalue=theme_options.index(self.theme) + 1)

            if selected_index is not None:
                selected_theme = theme_options[selected_index - 1]
                self.theme = selected_theme
                self.apply_theme()

        def cambiar_tamano_letra_action(self):
            # Acción de cambiar tamaño de letra
            size = simpledialog.askinteger("Cambiar Tamaño de Letra", "Ingrese el tamaño de letra:")
            if size is not None:
                if size > 0:
                    self.font_size = size
                    self.apply_font()
                else:
                    messagebox.showwarning("Error", "Ingrese un tamaño de letra válido (mayor que 0).")

        def cambiar_tipo_letra_action(self):
            # Acción de cambiar tipo de letra
            font_options = ["Arial", "Times New Roman", "Courier New", "Verdana", "Tahoma"]
            options_string = "\n".join(f"{i + 1}. {font}" for i, font in enumerate(font_options))
            prompt = f"Seleccione el tipo de letra:\n{options_string}"
            selected_index = simpledialog.askinteger("Cambiar Tipo de Letra", prompt, minvalue=1, maxvalue=len(font_options), initialvalue=font_options.index(self.font_family) + 1)
        
            if selected_index is not None:
                selected_font = font_options[selected_index - 1]
                self.font_family = selected_font
                self.apply_font()

        def apply_theme(self):
            # Aplicación de tema visual
            if self.theme == "Oscuro":
                self.root.config(bg="black")
                self.text_area.config(bg="black", fg="green")
                self.line_number_area.config(bg="black", fg="green")
                self.status_bar.config(bg="black", fg="green")
            elif self.theme == "Claro":
                self.root.config(bg="lightblue")
                self.text_area.config(bg="lightblue", fg="black")
                self.line_number_area.config(bg="lightblue", fg="black")
                self.status_bar.config(bg="lightblue", fg="black")
            else:
                # Predeterminado
                self.root.config(bg="")
                self.text_area.config(bg="white", fg="black")
                self.line_number_area.config(bg="white", fg="black")
                self.status_bar.config(bg="white", fg="black")

        def apply_font(self):
            # Aplicación de tipo y tamaño de letra
            self.text_area.config(font=(self.font_family, self.font_size)) #  establece el tipo y tamaño de letra en función de las variables
            self.line_number_area.config(font=(self.font_family, self.font_size)) # ste método asegura que tanto el área de texto principal como el área de números de línea tengan el mismo tipo y tamaño de letra

        def informacion_action(self):
            # Mensaje de información con la descripción del programa.
            info_text = """
            El programa es un editor de texto simple desarrollado en Python con la biblioteca Tkinter.  
            
                Características y funcionalidades:

                - Interfaz gráfica: El editor de texto utiliza Tkinter para crear una interfaz gráfica de usuario con menús y áreas de texto.
                - Funciones básicas de edición: Incluye funciones comunes de edición de texto como abrir, guardar, guardar como, deshacer y rehacer.
                - Formato de texto: Permite al usuario cambiar el tipo y tamaño de letra, así como elegir entre temas de visualización (predeterminado, oscuro o claro).
                - Resaltado de línea: Al hacer clic en una línea en la columna numérica, resalta la línea correspondiente en el área de texto.
                - Actualización en tiempo real: Muestra información sobre la posición del cursor (línea y columna) en una barra de estado.
                - Scrollbar y números de línea: Proporciona barras de desplazamiento vertical y horizontal, así como una columna numérica para mostrar los números de línea.
                - Guardado con confirmación: Al intentar cerrar el programa con cambios no guardados, pregunta al usuario si desea guardar los cambios.

                El código del programa es modular y fácilmente comprensible, permitiendo la incorporación de nuevas características o la realización de modificaciones.
                """
            # Ventana emergente con la información detallada.
            messagebox.showinfo("Información", info_text)
            
        def Desarrollado_por_action(self):
            # Mensaje de Desarrollador.
            info_text = """
            Cristian Daniel Gomez Gonzalez. 
            Carnet: 7690-22-3929
            Curos Algoritmos 2023
            Ingeniero Melvin Cali
            Ingeniería en Sistemas 
            Universidad Mariano Gálvez de Guatemala
                """
            # Ventana emergente con la información del desarrollador detallada.
            messagebox.showinfo("Desarrollado por", info_text)        
            

    if __name__ == "__main__":
        # Crear la ventana principal de la aplicación y ejecutar el bucle principal.
        root = tk.Tk() # proporciona una ventana principal para la interfaz gráfica.
        app = TextEditorApp(root) # la ventana principal de la aplicación se integrará en la interfaz
        root.mainloop() # Inicia el bucle principal de la interfaz gráfica,  espera eventos de entrada del usuario, como clics de ratón y pulsaciones de teclas, y responde a ellos
    -----
### CODIGO FUENTE.
-----
    import tkinter as tk 
    from tkinter import filedialog, messagebox, simpledialog 
    import subprocess  

    class TextEditorApp:
        """Clase que representa la aplicación de editor de texto con GUI."""
        def __init__(self, root):
            self.root = root
            self.root.title("Monaco Editor")
            self.setup_gui()
            self.current_file = None
            self.font_family = "Arial"
            self.font_size = 12
            self.theme = "Predeterminado"
            self.bind_events()
            self.root.protocol("WM_DELETE_WINDOW", self.on_closing)
            self.text_area.configure(font=(self.font_family, self.font_size))
            self.line_number_area.configure(font=(self.font_family, self.font_size))
            self.update_line_numbers()

        def setup_gui(self):
            """Configura la interfaz gráfica de la aplicación."""
            self.create_widgets()

        def create_widgets(self):
            self.create_line_number_area()
            self.create_scrollbars()
            self.create_status_bar()
            self.create_text_area()
            self.create_menu()

        def create_line_number_area(self):
            self.line_number_area = tk.Text(self.root, width=4, wrap="none", takefocus=0)
            self.line_number_area.pack(side=tk.LEFT, fill=tk.Y)
            self.line_number_area.config(state=tk.DISABLED)

        def create_scrollbars(self):
            self.scrollbar_y = tk.Scrollbar(self.root, orient=tk.VERTICAL)
            self.scrollbar_y.pack(side=tk.RIGHT, fill=tk.Y)
            self.scrollbar_x = tk.Scrollbar(self.root, orient=tk.HORIZONTAL)
            self.scrollbar_x.pack(side=tk.BOTTOM, fill=tk.X)

        def create_status_bar(self):
            self.status_bar = tk.Label(self.root, text="Línea: 1 | Columna: 0", bd=1, relief=tk.SUNKEN, anchor=tk.W)
            self.status_bar.pack(side=tk.BOTTOM, fill=tk.X)

        def create_text_area(self):
            self.text_area = tk.Text(self.root, wrap="none", yscrollcommand=self.scrollbar_y.set, xscrollcommand=self.scrollbar_x.set, undo=True, autoseparators=True, maxundo=-1)
            self.text_area.pack(expand="yes", fill="both")
            self.text_area.bind("<Motion>", self.update_status_bar)
            self.text_area.bind("<KeyRelease>", self.update_status_bar)
            self.text_area.bind("<Configure>", self.update_scrollbars)
            self.text_area.bind("<MouseWheel>", self.on_mousewheel)
            self.text_area.bind("<Button-1>", self.resaltar_linea_action)
            self.scrollbar_y.config(command=self.text_area.yview)
            self.scrollbar_x.config(command=self.text_area.xview)
            self.text_area.bind("<Configure>", self.update_line_numbers)
            self.text_area.bind("<Control-r>", self.rehacer_action)
            self.text_area.bind("<Control-R>", self.rehacer_action)

        def create_menu(self):
            self.menu_bar = tk.Menu(self.root)
            self.root.config(menu=self.menu_bar)

            menus = {
                "Archivo": ["Abrir", "Guardar", "Guardar como", "-", "Salir"],
                "Editar": ["Deshacer/ Ctrl Z", "Rehacer/ Ctrl R", "-", "Copiar/ Ctrl C", "Pegar/ Ctrl V"],
                "Configuración": ["Cambiar tema", "Cambiar tamaño de letra", "Cambiar tipo de letra"],
                "Ayuda": ["Información", "Manual de Usuario", "Desarrollado por"]
            }

            for menu_name, menu_items in menus.items():
                self.create_menu_items(menu_name, menu_items)

        def create_menu_items(self, menu_name, menu_items):
            menu = tk.Menu(self.menu_bar, tearoff=0)
            self.menu_bar.add_cascade(label=menu_name, menu=menu)

            for item in menu_items:
                if item == "-":
                    menu.add_separator()
                elif item == "Información":
                    menu.add_command(label=item, command=self.informacion_action)
                elif item == "Manual de Usuario":
                    menu.add_command(label=item, command=self.abrir_manual_usuario)
                elif item == "Desarrollado por":
                    menu.add_command(label=item, command=self.Desarrollado_por_action)
                else:
                    if "tipo de letra" in item:
                        menu.add_command(label=item, command=self.cambiar_tipo_letra_action, accelerator="")
                    elif "tamaño de letra" in item:
                        menu.add_command(label=item, command=self.cambiar_tamano_letra_action, accelerator="")
                    else:
                        command_name = f"{item.lower().replace(' ', '_')}_action"
                        accelerator = getattr(self, f"{item.lower().replace(' ', '_')}_accelerator", "")
                        menu.add_command(label=item, command=getattr(self, command_name, lambda: None), accelerator=accelerator)
                    if accelerator:
                        self.root.bind(accelerator, getattr(self, command_name, lambda x: None))

        def abrir_manual_usuario(self):
            try:
                subprocess.Popen(["C:\Program Files\Adobe\Acrobat DC\Acrobat\Acrobat.exe", r"C:\Users\Cristian\Desktop\Curso ALGORITMOS 2023\Agoritmos en C++\Proyecto III\Manual de Usuario.pdf"])
            except FileNotFoundError:
                messagebox.showerror("Error", "No se pudo encontrar el archivo de PDF predeterminado. Por favor, abre el manual de usuario manualmente.")

        def abrir_action(self):
            print("Abriendo archivo")
            file_path = filedialog.askopenfilename(filetypes=[("Archivos de texto", "*.txt"), ("Todos los archivos", "*.*")])

            if file_path:
                with open(file_path, "r") as file:
                    content = file.read()
                    self.text_area.delete("1.0", tk.END)
                    self.text_area.insert(tk.END, content)

                self.current_file = file_path
                self.update_line_numbers()

        def guardar_action(self):
            if self.current_file:
                with open(self.current_file, "w") as file:
                    content = self.text_area.get("1.0", tk.END)
                    file.write(content)
            else:
                self.guardar_como_action()

        def guardar_como_action(self):
            print("Guardando archivo como")
            file_path = filedialog.asksaveasfilename(defaultextension=".txt", filetypes=[("Archivos de texto", "*.txt;*.cpp;*.cs;*.py"), ("Todos los archivos", "*.*")])

            if file_path:
                with open(file_path, "w") as file:
                    content = self.text_area.get("1.0", tk.END)
                    file.write(content)

                self.current_file = file_path
                self.update_line_numbers()

        def copiar_action(self, event=None):
            print("Copiando texto")
            self.text_area.event_generate("<<Copy>>")

        def pegar_action(self, event=None):
            print("Pegando texto")
            self.text_area.event_generate("<<Paste>>")

        def deshacer_action(self, event=None):
            print("Deshaciendo")
            try:
                if self.text_area.edit_modified():
                    self.text_area.edit_undo()
                    self.update_line_numbers()
            except tk.TclError:
                pass

        def rehacer_action(self, event=None):
            print("Rehaciendo")
            try:
                self.text_area.edit_redo()
                self.update_line_numbers()
            except tk.TclError:
                pass

        def update_status_bar(self, event=None):
            cursor_position = self.text_area.index(tk.CURRENT)
            line, column = cursor_position.split('.')
            status_text = f"Línea: {line} | Columna: {column}"
            self.status_bar.config(text=status_text)

        def update_scrollbars(self, event=None):
            self.scrollbar_y.set(0.0, tk.END)
            self.scrollbar_x.set(0.0, tk.END)

        def update_line_numbers(self, event=None):
            self.line_number_area.config(state=tk.NORMAL)
            self.line_number_area.delete("1.0", tk.END)
            num_lines = str(self.text_area.index(tk.END).split('.')[0])
            self.line_number_area.insert(tk.END, '\n'.join(str(i) for i in range(1, int(num_lines) + 1)))
            self.line_number_area.config(state=tk.DISABLED)

            last_line_num_digits = len(str(num_lines))
            self.line_number_area.config(width=last_line_num_digits)

        def on_mousewheel(self, event):
            self.line_number_area.yview_scroll(int(-1 * event.delta), "units")
            self.text_area.yview_scroll(int(-1 * event.delta), "units")

        def focus_in_event(self, event):
            self.update_line_numbers()

        def bind_events(self):
            self.text_area.bind("<FocusIn>", self.focus_in_event)
            self.text_area.bind("<FocusOut>", lambda event: None)

        def on_closing(self):
            if self.is_text_edited():
                save_option = messagebox.askyesnocancel("Cerrar", "¿Desea guardar los cambios antes de cerrar?")
                if save_option is not None:
                    if save_option:
                        self.guardar_action()
                    self.root.destroy()
            else:
                self.root.destroy()

        def is_text_edited(self):
            return self.text_area.edit_modified()

        def resaltar_linea_action(self, event):
            cursor_position = self.text_area.index(tk.CURRENT)
            line_number = cursor_position.split('.')[0]
            start_index = f"{line_number}.0"
            end_index = f"{line_number}.end"

            self.text_area.tag_remove("highlight", "1.0", tk.END)
            self.line_number_area.tag_remove("highlight", "1.0", tk.END)

            self.text_area.tag_add("highlight", start_index, end_index)
            self.line_number_area.tag_add("highlight", start_index, end_index)

            self.text_area.tag_configure("highlight", background="yellow")
            self.line_number_area.tag_configure("highlight", background="yellow")

        def cambiar_tema_action(self):
            theme_options = ["Predeterminado", "Oscuro", "Claro"]
            options_string = "\n".join(f"{i + 1}. {theme}" for i, theme in enumerate(theme_options))
            prompt = f"Seleccione el tema:\n{options_string}"
            selected_index = simpledialog.askinteger("Cambiar Tema", prompt, minvalue=1, maxvalue=len(theme_options), initialvalue=theme_options.index(self.theme) + 1)

            if selected_index is not None:
                selected_theme = theme_options[selected_index - 1]
                self.theme = selected_theme
                self.apply_theme()

        def cambiar_tamano_letra_action(self):
            size = simpledialog.askinteger("Cambiar Tamaño de Letra", "Ingrese el tamaño de letra:")
            if size is not None:
                if size > 0:
                    self.font_size = size
                    self.apply_font()
                else:
                    messagebox.showwarning("Error", "Ingrese un tamaño de letra válido (mayor que 0).")

        def cambiar_tipo_letra_action(self):
            font_options = ["Arial", "Times New Roman", "Courier New", "Verdana", "Tahoma"]
            options_string = "\n".join(f"{i + 1}. {font}" for i, font in enumerate(font_options))
            prompt = f"Seleccione el tipo de letra:\n{options_string}"
            selected_index = simpledialog.askinteger("Cambiar Tipo de Letra", prompt, minvalue=1, maxvalue=len(font_options), initialvalue=font_options.index(self.font_family) + 1)
        
            if selected_index is not None:
                selected_font = font_options[selected_index - 1]
                self.font_family = selected_font
                self.apply_font()

        def apply_theme(self):
            if self.theme == "Oscuro":
                self.root.config(bg="black")
                self.text_area.config(bg="black", fg="green")
                self.line_number_area.config(bg="black", fg="green")
                self.status_bar.config(bg="black", fg="green")
            elif self.theme == "Claro":
                self.root.config(bg="lightblue")
                self.text_area.config(bg="lightblue", fg="black")
                self.line_number_area.config(bg="lightblue", fg="black")
                self.status_bar.config(bg="lightblue", fg="black")
            else:
                self.root.config(bg="")
                self.text_area.config(bg="white", fg="black")
                self.line_number_area.config(bg="white", fg="black")
                self.status_bar.config(bg="white", fg="black")

        def apply_font(self):
            self.text_area.config(font=(self.font_family, self.font_size))
            self.line_number_area.config(font=(self.font_family, self.font_size))

        def informacion_action(self):
            info_text = """
            El programa es un editor de texto simple desarrollado en Python con la biblioteca Tkinter.  
            
                Características y funcionalidades:

                - Interfaz gráfica: El editor de texto utiliza Tkinter para crear una interfaz gráfica de usuario con menús y áreas de texto.
                - Funciones básicas de edición: Incluye funciones comunes de edición de texto como abrir, guardar, guardar como, deshacer y rehacer.
                - Formato de texto: Permite al usuario cambiar el tipo y tamaño de letra, así como elegir entre temas de visualización (predeterminado, oscuro o claro).
                - Resaltado de línea: Al hacer clic en una línea en la columna numérica, resalta la línea correspondiente en el área de texto.
                - Actualización en tiempo real: Muestra información sobre la posición del cursor (línea y columna) en una barra de estado.
                - Scrollbar y números de línea: Proporciona barras de desplazamiento vertical y horizontal, así como una columna numérica para mostrar los números de línea.
                - Guardado con confirmación: Al intentar cerrar el programa con cambios no guardados, pregunta al usuario si desea guardar los cambios.

                El código del programa es modular y fácilmente comprensible, permitiendo la incorporación de nuevas características o la realización de modificaciones.
                """
            messagebox.showinfo("Información", info_text)
            
        def Desarrollado_por_action-(self):
            info_text = """
            Cristian Daniel Gomez Gonzalez. 
            Carnet: 7690-22-3929
            Curos Algoritmos 2023
            Ingeniero Melvin Cali
            Ingeniería en Sistemas 
            Universidad Mariano Gálvez de Guatemala
                """
            messagebox.showinfo("Desarrollado por", info_text)         

    if __name__ == "__main__":
        root = tk.Tk()
        app = TextEditorApp(root) 
        root.mainloop() 
----
## Cristian Daniel Gomez Gonzalez
## Carnet: 7690-22-3929
## Universidad Mariano Galvez de Guatemala
## Ingeniero Melvin Cali
## Curso de ALgomritmos