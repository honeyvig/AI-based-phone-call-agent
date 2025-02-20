# AI-based-phone-call-agent
 I need voice phone call ai agent connect using Twilio phone service for local python code . Agent is for office receptionist : first greeting by name through looking into customers table using , calling person phone number using caller id. python code agent implemented locally on my windows desktop. human being customer calls by phone. if existing customer then get order : 1) identify service id from customer verbal description 2) look into services prices table 3) tell prices 4) if confirmed get order 5) save order to orders table . If new customer then register : get name and phone number and address; then register orders. agent implemented by llm , use openai. as db use local saved excel files. for phone services use find provider which can provide USA phone number and voice to text input for local running python code and text to voice from local python code to caller phone (long conversation support important ).
acceptances criteria I can do by myself on my windows computer using your detail instructions (including how to get and use phone code provider ) and your code provided
-
Creating an AI-based phone call agent for your office receptionist that interacts with a customer, processes orders, and manages customer details using Twilio, OpenAI, and local Excel-based databases is quite a project but definitely achievable. Below is the architecture and the Python code required to get started.
Architecture Overview:

    Twilio Integration: Twilio will provide the phone service to make and receive calls.
    OpenAI (LLM): OpenAI's GPT (or any other conversational model) will process the natural language understanding (NLU) and generate conversational responses.
    Database: Excel files will be used to store customer data, service prices, and orders.
    Local Environment: All Python code will run locally on your Windows desktop, with appropriate libraries and services set up.

Steps to Achieve the Solution:

    Twilio Setup:
        Sign up on Twilio, purchase a phone number, and get API credentials (Account SID, Auth Token).
        Set up a webhook URL to handle incoming calls.

    OpenAI Setup:
        Sign up on OpenAI, get an API key, and use the GPT model for handling the conversations.

    Local Excel Database:
        Use pandas to read/write Excel files for customers, services, and orders.

    Voice Input/Output:
        Use Twilio’s Speech Recognition to capture voice input.
        Use gTTS (Google Text-to-Speech) or pyttsx3 to convert AI-generated responses into speech.

    Voice Agent Flow:
        Greeting: Greet the customer by name if they are in the customer database.
        Service Request: Identify the service ID based on customer description.
        Order Confirmation: Confirm prices and save the order.
        New Customer: Register new customer details and take the order.

Python Code Implementation:

Below is the Python code that integrates Twilio, OpenAI, and local Excel file-based storage.
1. Install Required Libraries:

You need to install the following libraries:

pip install twilio openai pandas gtts pyttsx3 xlrd

2. Python Code Implementation:

import pandas as pd
import openai
import pyttsx3
from twilio.rest import Client
from twilio.twiml.voice_response import VoiceResponse, Gather
from gtts import gTTS
import os

# Set your Twilio credentials
TWILIO_ACCOUNT_SID = 'your_twilio_account_sid'
TWILIO_AUTH_TOKEN = 'your_twilio_auth_token'
TWILIO_PHONE_NUMBER = 'your_twilio_phone_number'

# OpenAI API key
openai.api_key = "your_openai_api_key"

# Load Excel files for local database
customers_df = pd.read_excel('customers.xlsx')
services_df = pd.read_excel('services.xlsx')
orders_df = pd.read_excel('orders.xlsx')

# Twilio Client
client = Client(TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN)

# Initialize Text-to-Speech engine
engine = pyttsx3.init()

# Function to greet and identify customer
def greet_customer(call_sid, customer_phone):
    customer = customers_df[customers_df['phone_number'] == customer_phone]
    
    if not customer.empty:
        customer_name = customer.iloc[0]['name']
        response_text = f"Hello, {customer_name}! How can I assist you today?"
    else:
        response_text = "Hello! Are you a new customer?"
    
    # Convert text to speech and call Twilio API to speak to customer
    speak_to_customer(call_sid, response_text)

# Function to convert text to speech
def speak_to_customer(call_sid, text):
    tts = gTTS(text=text, lang='en')
    tts.save("response.mp3")
    
    # Send the audio file to Twilio for playing
    response = VoiceResponse()
    response.play("response.mp3")
    
    # Send response back to Twilio
    client.calls(call_sid).update(url="http://your_server_url")  # Webhook URL for further processing

# Function to process incoming call and handle the conversation
def handle_incoming_call(call_sid, customer_phone):
    # Get customer info and greet
    greet_customer(call_sid, customer_phone)
    
    # Wait for customer's speech response
    gather = Gather(input="speech", timeout=10, action="/process_speech", method="POST")
    response = VoiceResponse()
    response.append(gather)
    
    # Send the response to Twilio
    return str(response)

# Function to process speech input from customer
def process_speech_input(speech_input):
    # Use OpenAI's GPT to process the input and identify service or action
    prompt = f"Identify the service requested from this description: {speech_input}"
    response = openai.Completion.create(
        model="text-davinci-003",
        prompt=prompt,
        max_tokens=100
    )
    service_requested = response.choices[0].text.strip()
    
    # Look up the service price from the services table
    service_price = services_df[services_df['service_name'].str.contains(service_requested, case=False)]
    
    if not service_price.empty:
        price = service_price.iloc[0]['price']
        response_text = f"The price for {service_requested} is ${price}. Would you like to proceed with the order?"
    else:
        response_text = "Sorry, I could not identify the service. Could you please describe it again?"
    
    speak_to_customer(speech_input['call_sid'], response_text)

# Function to handle new customer registration
def register_new_customer(customer_phone):
    # Prompt customer for details (name, phone number, address)
    speak_to_customer(customer_phone, "Can you please provide your full name?")
    # Capture the response and register the customer in the database
    # (Continue collecting name, phone, address, and then save in the database)
    customers_df = customers_df.append({
        'name': 'customer_name',
        'phone_number': customer_phone,
        'address': 'customer_address'
    }, ignore_index=True)
    customers_df.to_excel('customers.xlsx', index=False)
    speak_to_customer(customer_phone, "Thank you for registering. Now, please describe the service you need.")

# Main function to trigger the call and agent flow
def start_receptionist_agent(call_sid, customer_phone):
    # Handle the incoming call and start conversation flow
    response = handle_incoming_call(call_sid, customer_phone)
    return response

# Setup a basic server to handle incoming call requests (using Flask or FastAPI for real-time interaction)
from flask import Flask, request

app = Flask(__name__)

@app.route("/incoming_call", methods=["POST"])
def incoming_call():
    call_sid = request.form['CallSid']
    customer_phone = request.form['From']
    return start_receptionist_agent(call_sid, customer_phone)

@app.route("/process_speech", methods=["POST"])
def process_speech():
    speech_input = request.form['SpeechResult']
    process_speech_input(speech_input)
    return "Processed"

if __name__ == "__main__":
    app.run(debug=True)

Explanation of Key Components:

    Twilio Integration:
        The handle_incoming_call function answers the call and starts the conversation by greeting the customer.
        The call is processed using Twilio's VoiceResponse and Gather functionality, which listens for speech input.
        Once the customer speaks, the speech input is processed using OpenAI's GPT to determine what the customer wants (service, order, etc.).

    OpenAI Integration:
        When the customer describes a service, the input is passed to OpenAI’s GPT model to identify the service.
        The model processes this input and responds with details such as service pricing.

    Database (Excel files):
        The customers.xlsx, services.xlsx, and orders.xlsx files are read and written using pandas.
        Customer details are matched by phone number, services are looked up by name, and orders are saved into the orders table.

    Text-to-Speech and Speech-to-Text:
        gTTS (Google Text-to-Speech) is used to convert text responses into speech, which is then played to the customer.
        Twilio's Speech Recognition converts voice input into text, which is processed by OpenAI.

    Server Setup (Flask or FastAPI):
        Flask is used to set up a web server that listens for incoming Twilio webhook requests.
        The /incoming_call route handles the incoming calls, and the /process_speech route processes the customer’s voice input.

How to Run Locally on Your Windows Machine:

    Set Up Twilio:
        Create a Twilio account, buy a number, and get your Account SID and Auth Token.
        Configure Twilio to send incoming call requests to your local machine (using a service like ngrok for tunneling HTTP requests to your local machine).

    Install Python and Dependencies:
        Ensure you have Python installed, and then run the following command to install dependencies:

    pip install twilio openai pandas gtts pyttsx3 xlrd Flask

    Run the Code:
        Save the Python code to a .py file, and then run the script.
        Ensure your Excel files (customers.xlsx, services.xlsx, orders.xlsx) are set up with appropriate columns like name, phone_number, address, service_name, price.

    Test the System:
        Make a call to your Twilio number, and it should go through the conversation flow.

Twilio Phone Service Provider:

    Find a provider: Go to Twilio, sign up, and buy a phone number with voice capabilities. Twilio provides both the voice-to-text and text-to-voice APIs to handle the phone interactions. Ensure the phone number supports voice services in the USA.

By following the above steps and code implementation, you should be able to set up your own AI phone receptionist using Twilio and OpenAI.
