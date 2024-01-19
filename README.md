# Clima-Weather
Clima Weather: Real-time updates for cities. Developed with Kivy in Python, it retrieves data from OpenWeatherMap API, presenting current conditions and temperature. The user-friendly interface offers a Celsius/Fahrenheit dropdown, reflecting expertise in app development and API integration.

# Clima is an intuitive weather app created by Christian Beasley

from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.button import Button
from kivy.uix.dropdown import DropDown
import matplotlib.pyplot as plt
import requests

class ClimaApp(App):
    def build(self):
        self.title = "Clima"

        # GUI Design
        layout = BoxLayout(orientation="vertical", padding=10, spacing=10)
        self.city_input = TextInput(hint_text="Please enter your city", multiline=False)

        # unit dropdown switch between C & F
        unit_dropdown = DropDown()
        f_btn = Button(text="Fahrenheit", size_hint_y=None, height=44)
        c_btn = Button(text="Celsius", size_hint_y=None, height=44)
        f_btn.bind(on_release=lambda btn: self.select_unit(btn.text))
        c_btn.bind(on_release=lambda btn: self.select_unit(btn.text))
        unit_dropdown.add_widget(f_btn)
        unit_dropdown.add_widget(c_btn)

        self.unit_button = Button(text="Select Unit", size_hint=(None, None), size=(200, 44))
        self.unit_button.bind(on_release=unit_dropdown.open)
        unit_dropdown.bind(on_select=lambda instance, x: setattr(self.unit_button, 'text', x))

        get_weather_button = Button(text="Get Weather", on_press=self.get_weather)
        self.weather_label = Label(text="Weather: ")
        self.temp_label = Label(text="Temperature: ")

        # Widget Design
        layout.add_widget(self.city_input)
        layout.add_widget(self.unit_button)
        layout.add_widget(get_weather_button)
        layout.add_widget(self.weather_label)
        layout.add_widget(self.temp_label)
        return layout

    def get_weather(self, instance):

        city = self.city_input.text
        unit = "imperial" if self.unit_button.text == "Fahrenheit" else "metric"

        if city:
            try:
                api_key = "307aed4f5f901e695a2a5443c7c8c54c"
                weather_data = requests.get(
                    f"https://api.openweathermap.org/data/2.5/weather?q={city}&units={unit}&APPID={api_key}"
                )
                weather_json = weather_data.json()

                weather = weather_json["weather"][0]["main"]
                temp = round(weather_json["main"]["temp"])

                self.weather_label.text = f'Weather: {weather}'
                self.temp_label.text = f'Temperature: {temp}°F' if unit == "imperial" else f'Temperature: {temp}°C'

            except requests.exceptions.RequestException as e:
                self.weather_label.text = 'Error: Unable to fetch data.'
                self.temp_label.text = ''
        else:
            self.weather_label.text = 'Please enter a city name.'
            self.temp_label.text = ''

    def select_unit(self, selected_unit):
        self.unit_button.text = selected_unit

    def show_graph(self, instance):
        cities = self.city_input.text.split(",")
        unit = "imperial" if self.unit_button.text == "Fahrenheit" else "metric"
        temperatures = []

        try:
            for city in cities:
                if city:
                    api_key = "307aed4f5f901e695a2a5443c7c8c54c"
                    weather_data = requests.get(
                        f"https://api.openweathermap.org/data/2.5/weather?q={city}&units={unit}&APPID={api_key}"
                    )
                    weather_json = weather_data.json()

                    temp = round(weather_json["main"]["temp"])
                    temperatures.append(temp)

                else:
                    temperatures.append(None)

        except requests.exceptions.RequestException as e:
            temperatures.append(None)

        plt.bar(cities, temperatures, color='blue')
        plt.xlabel('Cities')
        plt.ylabel('Temperature')
        plt.title('Temperature in Cities')
        plt.show()

if __name__ == '__main__':
    ClimaApp().run()
