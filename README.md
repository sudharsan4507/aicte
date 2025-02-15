# aicte
ai powered healthcare chatbot for diabetes patients

import openai
import tkinter as tk
from tkinter import messagebox, scrolledtext
from tkinter import ttk
import pyttsx3
import speech_recognition as sr
import random

try:
    import micropip
except ModuleNotFoundError:
    messagebox.showerror("Module Error", "The 'micropip' module is missing. Please install the required dependencies.")
    exit()

# OpenAI API Key (Replace 'your-api-key' with your actual OpenAI API key)
openai.api_key = "your-api-key"

# Initialize text-to-speech engine
engine = pyttsx3.init()
engine.setProperty('rate', 150)  # Speed of speech
engine.setProperty('volume', 1.0)

# Speech recognition setup
def recognize_speech():
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        try:
            chat_display.insert(tk.END, "Listening...\n")
            recognizer.adjust_for_ambient_noise(source)
            audio = recognizer.listen(source)
            user_input = recognizer.recognize_google(audio)
            user_entry.delete(0, tk.END)
            user_entry.insert(0, user_input)
            chatbot_response()
        except sr.UnknownValueError:
            chat_display.insert(tk.END, "Could not understand audio.\n")
        except sr.RequestError:
            chat_display.insert(tk.END, "Speech service unavailable.\n")

# Get chatbot response
def get_openai_response(user_input):
    """Sends the user input to OpenAI's GPT model and retrieves a response."""
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[{"role": "system", "content": "You are a diabetes healthcare assistant."},
                      {"role": "user", "content": user_input}]
        )
        return response["choices"][0]["message"]["content"].strip()
    except Exception as e:
        return f"Error: {str(e)}"

# Handle chatbot interaction
def chatbot_response():
    user_input = user_entry.get()
    if user_input.lower() == "check glucose":
        response = f"Your current glucose level is {random.randint(80, 180)} mg/dL."
    else:
        response = get_openai_response(user_input)
    chat_display.insert(tk.END, "You: " + user_input + "\n")
    chat_display.insert(tk.END, "Bot: " + response + "\n\n")
    chat_display.yview(tk.END)
    user_entry.delete(0, tk.END)
    engine.say(response)
    engine.runAndWait()

# Request doctor appointment
def request_doctor():
    messagebox.showinfo("Doctor Consultation", "Your request has been sent to the doctor!")

# Toggle dark mode
def toggle_dark_mode():
    bg_color = "#2E2E2E" if root.cget("bg") == "#f0f0f0" else "#f0f0f0"
    fg_color = "#FFFFFF" if bg_color == "#2E2E2E" else "#000000"
    root.configure(bg=bg_color)
    chat_display.configure(bg=bg_color, fg=fg_color, insertbackground=fg_color)
    user_entry.configure(bg=bg_color, fg=fg_color, insertbackground=fg_color)

# GUI Setup
root = tk.Tk()
root.title("OpenAI Diabetes Chatbot")
root.geometry("600x700")
root.configure(bg="#f0f0f0")

frame = ttk.Frame(root, padding=10)
frame.pack(fill=tk.BOTH, expand=True)

chat_display = scrolledtext.ScrolledText(frame, height=15, width=70, wrap=tk.WORD, state='normal', font=("Arial", 12))
chat_display.pack(pady=10)

user_entry = ttk.Entry(frame, width=50, font=("Arial", 12))
user_entry.pack(pady=5)

button_frame = ttk.Frame(frame)
button_frame.pack()

send_button = ttk.Button(button_frame, text="Send", command=chatbot_response)
send_button.grid(row=0, column=0, padx=5, pady=5)

voice_button = ttk.Button(button_frame, text="🎙 Speak", command=recognize_speech)
voice_button.grid(row=0, column=1, padx=5, pady=5)

doctor_button = ttk.Button(button_frame, text="Request Doctor", command=request_doctor)
doctor_button.grid(row=0, column=2, padx=5, pady=5)

dark_mode_button = ttk.Button(button_frame, text="🌙 Dark Mode", command=toggle_dark_mode)
dark_mode_button.grid(row=0, column=3, padx=5, pady=5)

root.mainloop()
