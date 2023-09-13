# Python-Foundation-batch-Fall-23
import requests

def get_weather(city, api_key):
    base_url = "https://api.openweathermap.org/data/2.5/weather?"
    complete_url = f"{base_url}q={city}&appid={api_key}&units=metric"
    
    response = requests.get(complete_url)
    data = response.json()
    
    if data["cod"] != "404":
        main_data = data["main"]
        temperature = main_data["temp"]
        humidity = main_data["humidity"]
        weather_data = data["weather"]
        weather_description = weather_data[0]["description"]
        
        print(f"Weather in {city}:")
        print(f"Temperature: {temperature}°C")
        print(f"Humidity: {humidity}%")
        print(f"Description: {weather_description}")
    else:
        print("City not found")

if __name__ == "__main__":
    city_name = input("Enter city name: ")
    api_key = "YOUR_API_KEY"  # Replace with your OpenWeatherMap API key
    
    get_weather(city_name, api_key)
