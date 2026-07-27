# MedAwareApp
# An app to to raise awareness of syntomps or how to act in certain medical urgencies in a funny and easy way


import tkinter as tk
import webbrowser
from tkinter import ttk


class MedAwareApp:
    def __init__(self, root):
        self.root = root
        self.root.title("MedAware - Aprende jugando")
        self.root.geometry("1120x700")
        self.root.minsize(980, 640)
        self.root.configure(bg="#eef5ff")

        self.score = 0
        self.streak = 5
        self.level = 1
        self.correct_answers = 0
        self.total_answers = 0
        self.current_condition = "ACV"
        self.current_challenge = 0
        self.current_mode = "retos"
        self.scenario_index = 0

        style = ttk.Style()
        style.theme_use("clam")
        style.configure("Title.TLabel", font=("Segoe UI", 18, "bold"), foreground="#123b6d")
        style.configure("Small.TLabel", font=("Segoe UI", 10), foreground="#5d6d85")
        style.configure("Card.TFrame", background="#ffffff")
        style.configure("Sidebar.TFrame", background="#eef5ff")
        style.configure("Accent.TButton", font=("Segoe UI", 10, "bold"))
        style.configure("DashboardCard.TFrame", background="#f7f9ff")
        style.configure(
            "Lesson.TButton",
            font=("Segoe UI", 10),
            background="#f7f9ff",
            foreground="#123b6d"
        )
        style.map(
            "Lesson.TButton",
            background=[("active", "#e7f0ff"), ("pressed", "#d7e7ff")],
            foreground=[("active", "#123b6d")]
        )
        style.configure("Selected.TButton", font=("Segoe UI", 10, "bold"), background="#dfeeff")
        style.map(
            "Selected.TButton",
            background=[("active", "#cfe3ff"), ("pressed", "#bedaff")],
            foreground=[("active", "#123b6d")]
        )

        header = ttk.Frame(root, padding=(18, 14, 18, 8))
        header.pack(fill="x")

        ttk.Label(header, text="MedAware", style="Title.TLabel").pack(anchor="w")
        ttk.Label(
            header,
            text="Aprende a reconocer señales de alerta con retos cortos y divertidos. Esta es una app educativa, no busca reemplazar atención medica profesional.",
            style="Small.TLabel"
        ).pack(anchor="w", pady=(4, 0))

        stats = ttk.Frame(root, padding=(18, 0, 18, 10))
        stats.pack(fill="x")

        self.score_var = tk.StringVar(value="⭐ 0 pts")
        self.level_var = tk.StringVar(value="🏆 Nivel 1")
        self.streak_var = tk.StringVar(value="🔥 Racha 5")

        ttk.Label(stats, textvariable=self.score_var, foreground="#f39c12", font=("Segoe UI", 10, "bold")).pack(side="left")
        ttk.Label(stats, textvariable=self.level_var, foreground="#0d7a3e", font=("Segoe UI", 10, "bold")).pack(side="left", padx=(12, 0))
        ttk.Label(stats, textvariable=self.streak_var, foreground="#d35400", font=("Segoe UI", 10, "bold")).pack(side="left", padx=(12, 0))

        self.progress = ttk.Progressbar(root, mode="determinate", length=300)
        self.progress.pack(fill="x", padx=18, pady=(0, 12))
        self.progress["value"] = 0

        self.nav_var = tk.StringVar(value="retos")
        nav = ttk.Frame(root, padding=(18, 0, 18, 0))
        nav.pack(fill="x")

        self.nav_buttons = {}
        for text, mode in [
            ("Retos", "retos"),
            ("Casos y simulación", "casos_simulacion"),
            ("Progreso", "progreso")
        ]:
            btn = ttk.Button(
                nav,
                text=text,
                command=lambda m=mode: self.set_mode(m)
            )
            btn.pack(side="left", padx=(0, 6))
            self.nav_buttons[mode] = btn

        self.nav_info = ttk.Label(
            nav,
            text="",
            foreground="#5d6d85",
            font=("Segoe UI", 9)
        )
        self.nav_info.pack(side="right")

        main = ttk.Frame(root, padding=(18, 0, 18, 18))
        main.pack(fill="both", expand=True)

        left = ttk.Frame(main, style="Sidebar.TFrame", padding=10)
        left.pack(side="left", fill="y")
        left.pack_propagate(False)
        left.configure(width=300)

        ttk.Label(
            left,
            text="📘 Lecciones",
            font=("Segoe UI", 11, "bold"),
            foreground="#123b6d"
        ).pack(anchor="w", pady=(0, 6))

        self.condition_summary = ttk.Label(
            left,
            text="",
            foreground="#123b6d",
            wraplength=240,
            justify="left"
        )
        self.condition_summary.pack(anchor="w", pady=(0, 6))

        self.conditions = {
            "ACV": {
                "emoji": "🧠",
                "title": "ACV (Accidente Cerebrovascular)",
                "color": "#2d6cdf"
            },
            "Infarto": {
                "emoji": "❤️",
                "title": "Infarto",
                "color": "#d94f4f"
            },
            "Desmayo": {
                "emoji": "🌀",
                "title": "Desmayo",
                "color": "#7a5af8"
            },
            "Convulsiones": {
                "emoji": "⚡",
                "title": "Convulsiones",
                "color": "#f5a623"
            }
        }

        self.lesson_buttons = {}
        for name in self.conditions:
            btn = ttk.Button(
                left,
                text=f"{self.conditions[name]['emoji']}  {name}",
                command=lambda n=name: self.select_topic(n),
                style="Lesson.TButton"
            )
            btn.pack(fill="x", pady=3)
            self.lesson_buttons[name] = btn

        info_panel = ttk.Frame(left, padding=(0, 10, 0, 0))
        info_panel.pack(fill="x")

        resources_frame = ttk.LabelFrame(info_panel, text="Más información", padding=10)
        resources_frame.pack(fill="x", pady=(0, 10))

        ttk.Label(
            resources_frame,
            text="Consulta recursos confiables para aprender más sobre señales de alerta y primeros auxilios.",
            foreground="#5d6d85",
            wraplength=240,
            justify="left"
        ).pack(anchor="w")

        self.resource_link = tk.Button(
            resources_frame,
            text="Abrir recursos relacionados",
            relief="flat",
            bd=0,
            bg="#eef5ff",
            fg="#1a73e8",
            activebackground="#dfeeff",
            activeforeground="#0b57d0",
            cursor="hand2",
            font=("Segoe UI", 10, "underline"),
            command=self.open_resources
        )
        self.resource_link.pack(anchor="w", pady=(6, 0))

        emergency_frame = ttk.LabelFrame(info_panel, text="Números de emergencia", padding=10)
        emergency_frame.pack(fill="x")

        ttk.Label(emergency_frame, text="País:", foreground="#123b6d").grid(row=0, column=0, sticky="w", padx=(0, 6), pady=4)
        ttk.Label(emergency_frame, text="Estado o región:", foreground="#123b6d").grid(row=1, column=0, sticky="w", padx=(0, 6), pady=4)

        self.country_var = tk.StringVar(value="México")
        self.country_combo = ttk.Combobox(
            emergency_frame,
            textvariable=self.country_var,
            values=["México", "España", "Colombia", "Argentina", "Estados Unidos"],
            state="readonly",
            width=18
        )
        self.country_combo.grid(row=0, column=1, sticky="w", pady=4)
        self.country_combo.bind("<<ComboboxSelected>>", self.update_emergency_info)

        self.state_var = tk.StringVar()
        self.state_combo = ttk.Combobox(
            emergency_frame,
            textvariable=self.state_var,
            state="readonly",
            width=22
        )
        self.state_combo.grid(row=1, column=1, sticky="w", pady=4)
        self.state_combo.bind("<<ComboboxSelected>>", self.update_emergency_info)

        self.emergency_info = tk.StringVar(value="Selecciona un país y región para ver el número de emergencia.")
        ttk.Label(
            emergency_frame,
            textvariable=self.emergency_info,
            foreground="#123b6d",
            wraplength=240,
            justify="left"
        ).grid(row=2, column=0, columnspan=2, sticky="w", pady=(6, 0))

        self.emergency_numbers = {
            "México": {
                "CDMX": "911",
                "Jalisco": "911",
                "Nuevo León": "911",
                "Veracruz": "911",
                "Otro": "911"
            },
            "España": {
                "Madrid": "112",
                "Cataluña": "112",
                "Andalucía": "112",
                "Otro": "112"
            },
            "Colombia": {
                "Bogotá": "123",
                "Antioquia": "123",
                "Valle del Cauca": "123",
                "Otro": "123"
            },
            "Argentina": {
                "CABA": "911",
                "Buenos Aires": "911",
                "Córdoba": "911",
                "Otro": "911"
            },
            "Estados Unidos": {
                "California": "911",
                "Texas": "911",
                "Florida": "911",
                "Otro": "911"
            },
            "Union Europea": {
                "todos": "112"
            },
            "Reino Unido": {
                "todos": "999"
            },
            "Oceanía": {
                "Nueva Zelanda": "111",
                "Australia": "000"
            },
            "Asia": {
                "Japón": "119",
                "China": "120",
                "India": "112"
            },
            "Cuba": {
                "todos": "104"
            }
        }
        self.country_combo.config(values=list(self.emergency_numbers.keys()))
        self.update_emergency_info()

        right = ttk.Frame(main, style="Card.TFrame", padding=16)
        right.pack(side="left", fill="both", expand=True)

        self.challenge_card = ttk.Frame(right, style="Card.TFrame", padding=16)
        self.challenge_card.pack(fill="both", expand=True)

        self.section_banner = ttk.Frame(self.challenge_card)
        self.section_banner.pack(fill="x", pady=(0, 8))

        self.topic_label = ttk.Label(self.challenge_card, font=("Segoe UI", 14, "bold"), foreground="#123b6d")
        self.topic_label.pack(anchor="w")

        self.challenge_label = ttk.Label(
            self.challenge_card,
            text="",
            font=("Segoe UI", 12, "bold"),
            wraplength=760,
            justify="left"
        )
        self.challenge_label.pack(anchor="w", pady=(10, 8))

        self.options_frame = ttk.Frame(self.challenge_card)
        self.options_frame.pack(fill="x", pady=(6, 10))

        self.option_buttons = []
        for i in range(4):
            btn = ttk.Button(
                self.options_frame,
                text="",
                command=lambda i=i: self.check_answer(i)
            )
            btn.pack(fill="x", pady=3)
            self.option_buttons.append(btn)

        self.feedback_label = ttk.Label(
            self.challenge_card,
            text="",
            foreground="#0d7a3e",
            wraplength=760,
            justify="left"
        )
        self.feedback_label.pack(anchor="w")

        self.explanation_label = ttk.Label(
            self.challenge_card,
            text="",
            foreground="#5d6d85",
            wraplength=760,
            justify="left"
        )
        self.explanation_label.pack(anchor="w", pady=(6, 10))

        bottom_buttons = ttk.Frame(right)
        bottom_buttons.pack(fill="x", pady=(10, 0))

        ttk.Button(bottom_buttons, text="Siguiente reto", command=self.next_challenge).pack(side="left")
        ttk.Button(bottom_buttons, text="Mostrar explicación", command=self.show_explanation).pack(side="left", padx=(6, 0))

        self.content_label = ttk.Label(
            right,
            text="",
            foreground="#123b6d",
            wraplength=760,
            justify="left"
        )
        self.content_label.pack_forget()

        self.summary_text = tk.StringVar(value="")
        self.summary_label = ttk.Label(
            right,
            textvariable=self.summary_text,
            foreground="#123b6d",
            wraplength=760,
            justify="left"
        )
        self.summary_label.pack_forget()

        self.dashboard_frame = ttk.Frame(right, style="Card.TFrame")
        self.dashboard_frame.pack_forget()

        self.challenges = {
            "ACV": [
                {
                    "question": "¿Cuál de estas señales sugiere un posible ACV?",
                    "options": [
                        "Debilidad repentina en un lado del cuerpo",
                        "Dolor muscular después de entrenar",
                        "Picazón en la piel",
                        "Mareo leve después de levantarse"
                    ],
                    "answer": 0,
                    "explanation": "El ACV puede causar debilidad o adormecimiento repentino en un lado del cuerpo."
                },
                {
                    "question": "¿Qué acción es clave si crees que alguien puede estar teniendo un ACV?",
                    "options": [
                        "Esperar a ver si mejora solo",
                        "Llamar a emergencias y anotar cuándo comenzaron los síntomas",
                        "Dar agua",
                        "Dejarlo dormir"
                    ],
                    "answer": 1,
                    "explanation": "La atención rápida es muy importante. Anotar la hora ayuda al personal médico."
                }
            ],
            "Infarto": [
                {
                    "question": "¿Qué síntoma es muy común en un infarto?",
                    "options": [
                        "Dolor o presión en el pecho",
                        "Olor a café",
                        "Picazón en las piernas",
                        "Aumento de energía repentino"
                    ],
                    "answer": 0,
                    "explanation": "El dolor o presión en el pecho puede ser una señal importante de infarto."
                },
                {
                    "question": "Si una persona tiene dificultad para respirar y dolor en el pecho, ¿qué debes hacer primero?",
                    "options": [
                        "Pedirle que camine",
                        "Llamar a emergencias",
                        "Ofrecerle comida",
                        "Esperar 30 minutos"
                    ],
                    "answer": 1,
                    "explanation": "Si hay sospecha de infarto, la respuesta inmediata es buscar ayuda médica urgente."
                }
            ],
            "Desmayo": [
                {
                    "question": "¿Qué señal puede aparecer antes de un desmayo?",
                    "options": [
                        "Mareo o visión borrosa",
                        "Fiebre alta sin motivo",
                        "Masticar rápido",
                        "Hablar más fuerte"
                    ],
                    "answer": 0,
                    "explanation": "Antes de desmayarse, muchas personas sienten mareo o visión borrosa."
                },
                {
                    "question": "Después de que una persona se recupera de un desmayo, ¿qué es mejor?",
                    "options": [
                        "Hacerla levantarse rápido",
                        "Dejarla descansar y evitar levantarse de inmediato",
                        "Darle azúcar si no lo pide",
                        "Dejarla sola"
                    ],
                    "answer": 1,
                    "explanation": "Es mejor que la persona descanse y se levante con cuidado."
                }
            ],
            "Convulsiones": [
                {
                    "question": "¿Qué debes hacer si una persona tiene una convulsión?",
                    "options": [
                        "Poner algo en la boca",
                        "Alejar objetos peligrosos y girarla hacia un lado",
                        "Hacerla caminar",
                        "Darle agua"
                    ],
                    "answer": 1,
                    "explanation": "Protege el entorno y ayuda a mantener la vía aérea segura."
                },
                {
                    "question": "¿Cuándo conviene llamar a emergencias durante una convulsión?",
                    "options": [
                        "Si dura más de 5 minutos o se repite",
                        "Solo si la persona está riendo",
                        "Si el episodio dura menos de 10 segundos",
                        "Nunca"
                    ],
                    "answer": 0,
                    "explanation": "Un episodio largo o repetido requiere ayuda médica urgente."
                }
            ]
        }

        self.simulation_scenarios = [
            {
                "title": "Escena 1: Plaza con mareo",
                "question": "Una persona en una plaza se ve mareada, habla con dificultad y tiene una cara asimétrica. ¿Qué harías primero?",
                "options": [
                    "Esperar a ver si mejora",
                    "Llamar a emergencias y observar si puede hablar claramente",
                    "Pedirle que corra",
                    "Darle agua sin evaluar"
                ],
                "answer": 1,
                "explanation": "Si aparece debilidad o habla rara, lo correcto es pedir ayuda inmediata y evaluar la situación."
            },
            {
                "title": "Escena 2: Dolor en el pecho",
                "question": "Una persona reporta dolor fuerte en el pecho, sudoración y miedo. ¿Cuál es la mejor respuesta?",
                "options": [
                    "Pedirle que se acueste y espere",
                    "Llamar a emergencias y ayudarla a mantenerse tranquila",
                    "Ofrecerle comida rápida",
                    "Dejarla sola para que se calme"
                ],
                "answer": 1,
                "explanation": "El dolor intenso en el pecho con sudoración puede indicar un problema serio y requiere ayuda urgente."
            },
            {
                "title": "Escena 3: Desmayo después del sol",
                "question": "Un adolescente se desmaya después de estar mucho tiempo al sol. ¿Qué debes hacer primero?",
                "options": [
                    "Moverlo rápidamente a otro lugar",
                    "Asegurar el entorno, revisar respiración y dar apoyo si está consciente",
                    "Darle azúcar sin preguntar",
                    "Dejarlo sentado sin supervisión"
                ],
                "answer": 1,
                "explanation": "Lo primero es proteger la seguridad y evaluar la respuesta de la persona."
            },
            {
                "title": "Escena 4: Convulsión breve",
                "question": "Una persona empieza a convulsionar en la calle. ¿Qué acción es correcta?",
                "options": [
                    "Poner algo en la boca",
                    "Alejar objetos peligrosos y cuidar la cabeza",
                    "Hacerla caminar",
                    "Darle agua"
                ],
                "answer": 1,
                "explanation": "La prioridad es protegerla de lesiones y no introducir objetos en la boca."
            },
            {
                "title": "Escena 5: Reacción alérgica",
                "question": "Una persona dice que le cuesta respirar y tiene hinchazón en la cara después de comer algo. ¿Qué harías?",
                "options": [
                    "Esperar 10 minutos",
                    "Llamar a emergencias y buscar el medicamento de la persona si lo tiene",
                    "Pedirle que se acueste boca arriba",
                    "Dejarla sola"
                ],
                "answer": 1,
                "explanation": "La dificultad para respirar puede ser grave; pedir ayuda inmediata es clave."
            }
        ]

        self.correct_answer = None
        self.selected_answer = None
        self.select_topic("ACV")
        self.load_challenge()

    def set_mode(self, mode):
        self.current_mode = mode
        for key, btn in self.nav_buttons.items():
            btn.state(["!pressed"])
        self.nav_buttons[mode].state(["pressed"])
        mode_labels = {
            "retos": "Modo reto",
            "casos_simulacion": "Práctica clínica",
            "progreso": "Tu progreso",
            
        }
        self.nav_info.config(text=mode_labels.get(mode, ""))
        self.show_mode_content()

    def select_topic(self, topic):
        self.current_condition = topic
        for key, btn in self.lesson_buttons.items():
            btn.state(["!pressed"])
        self.lesson_buttons[topic].state(["pressed"])
        self.current_challenge = 0
        self.load_challenge()

    def load_challenge(self):
        challenge = self.challenges[self.current_condition][self.current_challenge]
        self.topic_label.config(text=f"{self.conditions[self.current_condition]['emoji']}  {self.current_condition} | Reto {self.current_challenge + 1}")
        self.challenge_label.config(text=challenge["question"])
        self.feedback_label.config(text="")
        self.explanation_label.config(text="")
        self.correct_answer = challenge["answer"]
        self.selected_answer = None
        self.condition_summary.config(
            text=f"{self.conditions[self.current_condition]['emoji']} {self.conditions[self.current_condition]['title']}"
        )

        for i, btn in enumerate(self.option_buttons):
            btn.config(text=challenge["options"][i])

    def check_answer(self, index):
        self.selected_answer = index
        if self.current_mode == "casos_simulacion":
            challenge = self.simulation_scenarios[self.scenario_index]
        else:
            challenge = self.challenges[self.current_condition][self.current_challenge]

        self.total_answers += 1
        if index == self.correct_answer:
            self.score += 10
            self.streak += 1
            self.correct_answers += 1
            self.feedback_label.config(text="✅ Correcto!", foreground="#0d7a3e")
            self.explanation_label.config(text=challenge["explanation"])
        else:
            self.streak = max(0, self.streak - 1)
            self.feedback_label.config(text="❌ Casi, pero no es esa.", foreground="#c0392b")
            self.explanation_label.config(text=f"La respuesta correcta es: {challenge['options'][self.correct_answer]}")

        self.score_var.set(f"⭐ {self.score} pts")
        self.streak_var.set(f"🔥 Racha {self.streak}")
        self.level = 1 + (self.score // 50)
        self.level_var.set(f"🏆 Nivel {self.level}")
        self.progress["value"] = min(100, self.score)

    def show_explanation(self):
        if self.current_mode == "casos_simulacion":
            challenge = self.simulation_scenarios[self.scenario_index]
        else:
            challenge = self.challenges[self.current_condition][self.current_challenge]
        self.explanation_label.config(text=challenge["explanation"])

    def load_simulation_scenario(self):
        scenario = self.simulation_scenarios[self.scenario_index]
        self.topic_label.config(text=f"🩺 Casos y simulación | Escena {self.scenario_index + 1}")
        self.challenge_label.config(text=scenario["question"])
        self.feedback_label.config(text="")
        self.explanation_label.config(text="")
        self.correct_answer = scenario["answer"]
        self.selected_answer = None
        self.condition_summary.config(text="Practica situaciones reales de emergencia")

        for i, btn in enumerate(self.option_buttons):
            btn.config(text=scenario["options"][i])

    def show_mode_content(self):
        if self.current_mode == "retos":
            self.challenge_card.pack(fill="both", expand=True)
            self.content_label.pack_forget()
            self.summary_label.pack_forget()
            self.dashboard_frame.pack_forget()
            self.load_challenge()
        elif self.current_mode == "casos_simulacion":
            self.challenge_card.pack(fill="both", expand=True)
            self.content_label.pack_forget()
            self.summary_label.pack_forget()
            self.dashboard_frame.pack_forget()
            self.load_simulation_scenario()
        elif self.current_mode == "progreso":
            self.challenge_card.pack_forget()
            self.content_label.pack_forget()
            self.summary_label.pack_forget()
            self.dashboard_frame.pack(fill="both", expand=True)
            self.show_progress_dashboard()
        else:
            self.challenge_card.pack_forget()
            self.content_label.pack(fill="both", expand=True)
            self.summary_label.pack(fill="x", pady=(0, 8))
            self.dashboard_frame.pack_forget()

    def show_progress_dashboard(self):
        for widget in self.dashboard_frame.winfo_children():
            widget.destroy()

        accuracy = 0 if self.total_answers == 0 else int((self.correct_answers / self.total_answers) * 100)
        progress_pct = min(100, self.score)

        header_bg = tk.Frame(self.dashboard_frame, bg="#eef5ff", height=80)
        header_bg.pack(fill="x")
        header_bg.pack_propagate(False)

        ttk.Label(
            header_bg,
            text="📈 Tu progreso",
            font=("Segoe UI", 16, "bold"),
            foreground="#123b6d"
        ).place(x=16, y=18)

        ttk.Label(
            header_bg,
            text=f"{self.correct_answers} respuestas correctas",
            foreground="#5d6d85",
            font=("Segoe UI", 10)
        ).place(x=16, y=48)

        cards_frame = ttk.Frame(self.dashboard_frame)
        cards_frame.pack(fill="x", padx=16, pady=(12, 10))

        cards = [
            ("⭐", "Puntos", f"{self.score}", "#f39c12"),
            ("🏆", "Nivel", f"{self.level}", "#0d7a3e"),
            ("🔥", "Racha", f"{self.streak}", "#d35400"),
            ("📊", "Precisión", f"{accuracy}%", "#1a73e8")
        ]

        for icon, label, value, color in cards:
            card = tk.Frame(cards_frame, bg="#f7f9ff", bd=1, relief="solid", padx=12, pady=10)
            card.pack(side="left", fill="y", padx=(0, 10))
            tk.Label(card, text=icon, bg="#f7f9ff", font=("Segoe UI", 18)).pack(anchor="w")
            tk.Label(card, text=label, bg="#f7f9ff", fg="#5d6d85", font=("Segoe UI", 9)).pack(anchor="w")
            tk.Label(card, text=value, bg="#f7f9ff", fg=color, font=("Segoe UI", 12, "bold")).pack(anchor="w")

        ttk.Label(
            self.dashboard_frame,
            text="Logros desbloqueados",
            foreground="#123b6d",
            font=("Segoe UI", 11, "bold")
        ).pack(anchor="w", padx=16, pady=(6, 8))

        achievements = [
            ("🏆", "Primer reto completado", self.total_answers >= 1),
            ("🔥", "Racha activa", self.streak >= 5),
            ("🧠", "Detective de emergencias", self.correct_answers >= 3),
            ("🩺", "Mentor de primeros auxilios", self.score >= 30)
        ]

        for emoji, text, unlocked in achievements:
            status = "✓" if unlocked else "•"
            color = "#0d7a3e" if unlocked else "#5d6d85"
            ttk.Label(
                self.dashboard_frame,
                text=f"{status} {emoji} {text}",
                foreground=color,
                font=("Segoe UI", 10)
            ).pack(anchor="w", padx=24, pady=2)

        ttk.Label(
            self.dashboard_frame,
            text="Progreso general",
            foreground="#123b6d",
            font=("Segoe UI", 11, "bold")
        ).pack(anchor="w", padx=16, pady=(12, 6))

        progress_bar = ttk.Progressbar(
            self.dashboard_frame,
            mode="determinate",
            length=700
        )
        progress_bar.pack(fill="x", padx=16, pady=(0, 4))
        progress_bar["value"] = progress_pct

        ttk.Label(
            self.dashboard_frame,
            text=f"{progress_pct}% completado",
            foreground="#5d6d85",
            font=("Segoe UI", 9)
        ).pack(anchor="w", padx=16)

    def open_resources(self):
        if getattr(self, "resources_window", None) and self.resources_window.winfo_exists():
            self.resources_window.lift()
            self.resources_window.focus_set()
            return

        self.resources_window = tk.Toplevel(self.root)
        self.resources_window.title("Recursos de apoyo")
        self.resources_window.geometry("420x260")
        self.resources_window.configure(bg="#f7f9ff")

        ttk.Label(
            self.resources_window,
            text="Recursos confiables",
            style="Title.TLabel"
        ).pack(anchor="w", padx=16, pady=(16, 6))

        ttk.Label(
            self.resources_window,
            text="Abre sitios recomendados para aprender más sobre emergencias médicas.",
            foreground="#5d6d85",
            wraplength=380,
            justify="left"
        ).pack(anchor="w", padx=16, pady=(0, 10))

        resources = [
            ("OMS - Salud mundial", "https://www.who.int/es"),
            ("CDC - Prevención y salud", "https://www.cdc.gov/"),
            ("Ministerio de Salud", "https://www.minsalud.gov.co/")
        ]

        for label, url in resources:
            ttk.Button(
                self.resources_window,
                text=label,
                command=lambda u=url: webbrowser.open(u)
            ).pack(fill="x", padx=16, pady=4)

    def update_emergency_info(self, event=None):
        country = self.country_var.get()
        regions = self.emergency_numbers.get(country, {})
        if not regions:
            self.emergency_info.set("No hay información disponible para ese país.")
            return

        region_names = list(regions.keys())
        self.state_combo.config(values=region_names)

        if self.state_var.get() not in region_names:
            self.state_var.set(region_names[0])

        region = self.state_var.get()
        number = regions.get(region, region_names[0])
        self.emergency_info.set(
            f"Número de emergencia en {region} ({country}): {number}"
        )

    def next_challenge(self):
        if self.current_mode == "casos_simulacion":
            self.scenario_index = (self.scenario_index + 1) % len(self.simulation_scenarios)
            self.load_simulation_scenario()
        else:
            self.current_challenge = (self.current_challenge + 1) % len(self.challenges[self.current_condition])
            self.load_challenge()


if __name__ == "__main__":
    root = tk.Tk()
    app = MedAwareApp(root)
    root.bind("<Return>", lambda event: app.next_challenge())
    root.bind("<F1>", lambda event: app.open_resources())
    root.mainloop()
