import pyttsx3
import datetime
import speech_recognition as sr
import wikipedia
import os
import webbrowser
import pyjokes
import pywhatkit as kit
from tkinter import *
from PIL import Image, ImageTk
from threading import Thread

# Styling
BG_COLOR = "#1A1A2E"
BUTTON_COLOR = "#00FFC6"
BUTTON_HOVER_COLOR = "#00BFA6"
BUTTON_FOREGROUND = "#000000"
BUTTON_FONT = ("Orbitron", 12, "bold")
HEADING_FONT = ("Orbitron", 24, "bold")
INSTRUCTION_FONT = ("Orbitron", 14)

# Voice setup
engine = pyttsx3.init('sapi5')
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[0].id)

entry = None
stop_flag = False

def speak(audio):
    engine.say(audio)
    engine.runAndWait()

def wish_time():
    global entry
    name = entry.get()
    hour = int(datetime.datetime.now().hour)
    if 0 <= hour < 6:
        speak('Good night! Sleep tight.')
    elif 6 <= hour < 12:
        speak('Good morning!')
    elif 12 <= hour < 18:
        speak('Good afternoon!')
    else:
        speak('Good evening!')
    speak(f"{name}, how can I help you?")

def take_command():
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        speak("Say something")
        recognizer.pause_threshold = 0.8
        recognizer.adjust_for_ambient_noise(source, duration=0.5)
        audio = recognizer.listen(source)
    try:
        query = recognizer.recognize_google(audio, language='en-in')
        return query
    except:
        speak("Please say again.")
        return "None"

def perform_task():
    global stop_flag
    while not stop_flag:
        query = take_command().lower()
        if 'wikipedia' in query:
            speak('Searching Wikipedia.')
            query = query.replace("wikipedia", "")
            try:
                results = wikipedia.summary(query, sentences=2)
                speak("According to Wikipedia")
                speak(results)
            except:
                speak("Could not find any result.")
        elif 'play' in query:
            song = query.replace('play', "")
            speak("Playing " + song)
            kit.playonyt(song)
        elif 'search' in query:
            s = query.replace('search', '')
            kit.search(s)
        elif 'open youtube' in query:
            webbrowser.open("https://www.youtube.com/")
        elif 'open google' in query:
            webbrowser.open("https://www.google.com/")
        elif 'the time' in query:
            str_time = datetime.datetime.now().strftime("%H:%M:%S")
            speak(f"The time is {str_time}")
        elif 'open code' in query:
            os.startfile(r"C:\Users\DELL\AppData\Local\Programs\Microsoft VS Code\Code.exe")
        elif 'joke' in query:
            speak(pyjokes.get_joke())
        elif "where is" in query:
            location = query.replace("where is", "")
            speak("Locating " + location)
            webbrowser.open("https://www.google.com/maps/place/" + location.replace(" ", "+"))
        elif 'exit' in query:
            speak("Thanks for your time. Goodbye.")
            stop_voice_assistant()

def stop_voice_assistant():
    global stop_flag
    speak("Stopping the Voice Assistant.")
    stop_flag = True

def start_voice_assistant():
    global stop_flag
    stop_flag = False
    wish_time()
    perform_task()

def on_enter(e):
    e.widget['background'] = BUTTON_HOVER_COLOR

def on_leave(e):
    e.widget['background'] = BUTTON_COLOR

def main():
    root = Tk()
    root.title("Gemini Voice Assistant")
    root.geometry("600x750")
    root.configure(bg=BG_COLOR)

    try:
        bg_img = Image.open("wallpaperflare.com_wallpaper.jpg")
        bg_img = bg_img.resize((600, 750), Image.ANTIALIAS)
        bg_photo = ImageTk.PhotoImage(bg_img)
        bg_label = Label(root, image=bg_photo)
        bg_label.image = bg_photo
        bg_label.place(x=0, y=0, relwidth=1, relheight=1)
    except:
        pass

    Label(root, text="Gemini Voice Assistant", font=HEADING_FONT,
          bg=BG_COLOR, fg=BUTTON_COLOR).pack(pady=30)

    global entry
    name_frame = Frame(root, bg=BG_COLOR)
    name_frame.pack(pady=10)
    Label(name_frame, text="Enter Your Name:", font=INSTRUCTION_FONT, bg=BG_COLOR, fg="white").pack(side="left", padx=10)
    entry = Entry(name_frame, font=("Arial", 12), width=30)
    entry.pack()

    Label(root, text="Press the button to start the assistant",
          font=INSTRUCTION_FONT, bg=BG_COLOR, fg="white").pack(pady=20)

    start_btn = Button(root, text="Start Voice Assistant", font=BUTTON_FONT,
                       bg=BUTTON_COLOR, fg=BUTTON_FOREGROUND, padx=20, pady=10,
                       command=lambda: Thread(target=start_voice_assistant).start())
    start_btn.pack(pady=15)
    start_btn.bind("<Enter>", on_enter)
    start_btn.bind("<Leave>", on_leave)

    extra_frame = Frame(root, bg=BG_COLOR)
    extra_frame.pack(pady=30)

    def create_button(text, cmd):
        btn = Button(extra_frame, text=text, font=BUTTON_FONT, bg=BUTTON_COLOR,
                     fg=BUTTON_FOREGROUND, padx=20, pady=10, command=cmd)
        btn.pack(pady=8)
        btn.bind("<Enter>", on_enter)
        btn.bind("<Leave>", on_leave)

    def get_weather():
        speak("Opening weather forecast.")
        webbrowser.open("https://www.weather.com/")

    def tell_joke():
        speak(pyjokes.get_joke())

    def open_youtube():
        speak("Opening YouTube.")
        webbrowser.open("https://www.youtube.com/")

    def open_google():
        speak("Opening Google.")
        webbrowser.open("https://www.google.com/")

    def exit_assistant():
        speak("Goodbye.")
        stop_voice_assistant()
        root.destroy()

    create_button("Check Weather", get_weather)
    create_button("Tell me a Joke", tell_joke)
    create_button("Open YouTube", open_youtube)
    create_button("Open Google", open_google)
    create_button("Exit Assistant", exit_assistant)

    root.mainloop()

if __name__ == "__main__":
    main()

