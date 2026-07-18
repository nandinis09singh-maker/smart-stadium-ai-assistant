# smart-stadium-ai-assistant
GenAI-powered Smart Stadium Assistant using Gemini AI, Arduino Uno, LCD, Ultrasonic Sensor, and Servo Motor.
# ==========================================================
# StadiumSense AI
# GenAI Smart Stadium Assistant
# Part 1
# ==========================================================

import os
import time
import serial
import pyttsx3
import speech_recognition as sr
import google.generativeai as genai

# -----------------------------
# CONFIGURATION
# -----------------------------

GEMINI_API_KEY = "PASTE_YOUR_GEMINI_API_KEY"

SERIAL_PORT = "COM3"      # Change if needed
BAUD_RATE = 9600

genai.configure(api_key=GEMINI_API_KEY)

model = genai.GenerativeModel("gemini-1.5-flash")

recognizer = sr.Recognizer()

engine = pyttsx3.init()

engine.setProperty("rate",170)

engine.setProperty("volume",1.0)

# -----------------------------
# Connect Arduino
# -----------------------------

try:
    arduino = serial.Serial(
        SERIAL_PORT,
        BAUD_RATE,
        timeout=1
    )

    time.sleep(2)

    print("Arduino Connected")

except:

    arduino = None

    print("Arduino Not Connected")

# -----------------------------
# Speak Function
# -----------------------------

def speak(text):

    print("\nAI :",text)

    engine.say(text)

    engine.runAndWait()

# -----------------------------
# Listen Function
# -----------------------------

def listen():

    with sr.Microphone() as source:

        recognizer.adjust_for_ambient_noise(source)

        print("\nListening...")

        try:

            audio = recognizer.listen(
                source,
                timeout=5,
                phrase_time_limit=8
            )

            command = recognizer.recognize_google(audio)

            print("You :",command)

            return command.lower()

        except:

            return ""

# -----------------------------
# Send Command to Arduino
# -----------------------------

def send(command):

    if arduino:

        arduino.write((command+"\n").encode())

# -----------------------------
# LCD Messages
# -----------------------------

def lcd(message):

    send("LCD:"+message)

# -----------------------------
# Servo Commands
# -----------------------------

def point_gate():

    send("GATE")

def point_food():

    send("FOOD")

def point_exit():

    send("EXIT")

def point_washroom():

    send("WASHROOM")

def point_help():

    send("HELP")

# -----------------------------
# Welcome
# -----------------------------

def welcome():

    lcd("WELCOME")

    speak(# ==========================================================
# GEMINI AI
# ==========================================================

SYSTEM_PROMPT = """
You are StadiumSense AI.

You are an intelligent AI assistant for the FIFA World Cup 2026.

Help visitors with:

• Gate information
• Stadium navigation
• Seat directions
• Food court locations
• Washrooms
• Parking
• Ticket guidance
• Accessibility services
• Medical assistance
• Emergency exits

Always give short, friendly and clear answers.

If someone reports an emergency, immediately advise them to contact the nearest stadium official and proceed to the nearest medical or emergency exit.

Never answer harmful requests.
"""

chat = model.start_chat(history=[])


def ask_gemini(question):

    prompt = SYSTEM_PROMPT + "\n\nVisitor: " + question

    try:

        response = chat.send_message(prompt)

        return response.text.strip()

    except Exception as e:

        print(e)

        return "Sorry, I am unable to answer right now."


# ==========================================================
# STADIUM KNOWLEDGE
# ==========================================================

gate_locations = {
    "gate 1": "North Entrance",
    "gate 2": "East Entrance",
    "gate 3": "South Entrance",
    "gate 4": "West Entrance"
}

food_courts = {
    "pizza":"Food Court A",
    "burger":"Food Court B",
    "coffee":"Food Court C",
    "drinks":"Food Court D"
}

washrooms = {
    "north":"Near Gate 1",
    "east":"Near Gate 2",
    "south":"Near Gate 3",
    "west":"Near Gate 4"
}


# ==========================================================
# COMMAND HANDLER
# ==========================================================

def handle_command(command):

    lcd("PROCESSING")

    # ---------------- GATE ----------------

    if "gate" in command:

        point_gate()

        answer = ask_gemini(command)

        lcd("GATE INFO")

        speak(answer)

        return

    # ---------------- FOOD ----------------

    if (
        "food" in command
        or "pizza" in command
        or "burger" in command
        or "coffee" in command
    ):

        point_food()

        answer = ask_gemini(command)

        lcd("FOOD COURT")

        speak(answer)

        return

    # ---------------- WASHROOM ----------------

    if (
        "washroom" in command
        or "toilet" in command
        or "restroom" in command
    ):

        point_washroom()

        answer = ask_gemini(command)

        lcd("WASHROOM")

        speak(answer)

        return

    # ---------------- PARKING ----------------

    if "parking" in command:

        answer = ask_gemini(command)

        lcd("PARKING")

        speak(answer)

        return

    # ---------------- TICKET ----------------

    if "ticket" in command:

        answer = ask_gemini(command)

        lcd("TICKET")

        speak(answer)

        return

    # ---------------- SEAT ----------------

    if (
        "seat" in command
        or "section" in command
        or "row" in command
    ):

        answer = ask_gemini(command)

        lcd("SEAT GUIDE")

        speak(answer)

        return

    # ---------------- MEDICAL ----------------

    if (
        "medical" in command
        or "doctor" in command
        or "first aid" in command
    ):

        point_help()

        lcd("MEDICAL")

        speak(
            "Please proceed to the nearest medical assistance center."
        )

        return

    # ---------------- EMERGENCY ----------------

    if (
        "fire" in command
        or "emergency" in command
        or "help" in command
        or "accident" in command
    ):

        point_exit()

        lcd("EMERGENCY")

        speak(
            "Emergency detected. Please move calmly to the nearest exit and contact stadium staff immediately."
        )

        return

    # ---------------- DEFAULT ----------------

    answer = ask_gemini(command)

    lcd("AI RESPONSE")

    speak(answer)# ==========================================================
# ARDUINO LISTENER
# ==========================================================

def check_arduino():

    if arduino is None:
        return

    try:

        if arduino.in_waiting:

            message = (
                arduino.readline()
                .decode()
                .strip()
            )

            print("Arduino :", message)

            if message == "PERSON":

                welcome()

    except Exception as e:

        print(e)


# ==========================================================
# STARTUP
# ==========================================================

def startup():

    print("=" * 50)
    print("      StadiumSense AI")
    print(" GenAI Smart Stadium Assistant")
    print("=" * 50)

    lcd("SYSTEM READY")

    speak(
        "System initialized successfully."
    )

    speak(
        "Waiting for visitors."
    )


# ==========================================================
# GOODBYE
# ==========================================================

def goodbye():

    lcd("THANK YOU")

    speak(
        "Thank you for visiting. Have a wonderful match."
    )


# ==========================================================
# MAIN LOOP
# ==========================================================

def main():

    startup()

    while True:

        # Check if Arduino detected a visitor
        check_arduino()

        command = listen()

        if command == "":
            continue

        # Exit program
        if (
            command == "exit"
            or command == "quit"
            or command == "stop"
        ):

            goodbye()

            break

        # Greeting
        if (
            "hello" in command
            or "hi" in command
        ):

            lcd("WELCOME")

            speak(
                "Hello! How may I assist you today?"
            )

            continue

        # Thanks
        if (
            "thank you" in command
            or "thanks" in command
        ):

            goodbye()

            continue

        # Handle visitor query
        handle_command(command)


# ==========================================================
# RUN PROGRAM
# ==========================================================

if __name__ == "__main__":

    try:

        main()

    except KeyboardInterrupt:

        print("\nProgram Closed")

        if arduino:
            arduino.close()
        "Welcome to the FIFA World Cup Smart Stadium. "
        "How may I help you today?"
    )
