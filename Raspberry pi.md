```bash
sudo raspi-config
```

https://randomnerdtutorials.com/raspberry-pi-ds18b20-python/

**Cofigure Nano file :**
```bash
sudo nano /boot/config.txt

```


**Write this line in nano**
```makefile
dtoverlay=w1-gpio,gpiopin=4

```


**Reload Modules**


```bash
sudo modprobe -r w1_therm
sudo modprobe -r w1_gpio
sudo modprobe w1_gpio
sudo modprobe w1_therm

```



**Check Sensor Address**: 
```bash
cd /sys/bus/w1/devices
ls

```

Now change directory to DS18B20 sensor
```shell
cd 28-00000f3fd9a2
```
 **cat** command to read the contents of the w1_slave file.
```shell
cat w1_slave
```





# **Display Temperature in Touch Screen Display**


**Install pyGame:**

```bash
sudo apt update
sudo apt install python3-pygame

```

```python
import os
import glob
import time
import pygame
from pygame.locals import *

# Initialize Pygame
pygame.init()

# Set up display (adjust resolution to match your display)
screen = pygame.display.set_mode((1024, 600), pygame.FULLSCREEN)

# Set a larger font size and create a font object
font = pygame.font.Font(None, 150)  # Increase font size to 150

# Define colors (using RGB values)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
BLUE = (0, 0, 255)

# Initialize temperature sensor
os.system('modprobe w1-gpio')
os.system('modprobe w1-therm')

base_dir = '/sys/bus/w1/devices/'
device_folder = glob.glob(base_dir + '28*')[0]
device_file = device_folder + '/w1_slave'

# Function to read raw temperature data from the sensor
def read_temp_raw():
    with open(device_file, 'r') as f:
        lines = f.readlines()
    return lines

# Function to extract the temperature in Celsius and Fahrenheit
def read_temp():
    lines = read_temp_raw()
    while lines[0].strip()[-3:] != 'YES':
        time.sleep(0.2)
        lines = read_temp_raw()
    equals_pos = lines[1].find('t=')
    if equals_pos != -1:
        temp_string = lines[1][equals_pos+2:]
        temp_c = float(temp_string) / 1000.0
        temp_f = temp_c * 9.0 / 5.0 + 32.0
        return temp_c, temp_f
    return None, None

# Main loop to display temperature continuously
running = True
while running:
    # Event handling to exit on ESC key press
    for event in pygame.event.get():
        if event.type == QUIT:
            running = False
        elif event.type == KEYDOWN:
            if event.key == K_ESCAPE:  # Exit on pressing ESC
                running = False

    # Read temperature
    temperature_celsius, temperature_fahrenheit = read_temp()

    # Fill the screen with a color (black)
    screen.fill((0, 0, 0))

    # Create the text surfaces with custom colors
    temp_c_text = font.render(f'{temperature_celsius:.2f} C', True, RED)  # Red for Celsius
    temp_f_text = font.render(f'{temperature_fahrenheit:.2f} F', True, BLUE)  # Blue for Fahrenheit


    # Create the text surfaces for Celsius and Fahrenheit
    temp_c_text = font.render(f'Temp: {temperature_celsius:.2f} C', True, (255, 255, 255))
    temp_f_text = font.render(f'Temp: {temperature_fahrenheit:.2f} F', True, (255, 255, 255))

    # Display the text on the screen at specified positions
    screen.blit(temp_c_text, (100, 150))  # Adjust coordinates as needed
    screen.blit(temp_f_text, (100, 350))  # Adjust coordinates as needed

    # Update the display
    pygame.display.flip()

    # Delay to control the update rate (e.g., 1 second between readings)
    time.sleep(1)

# Quit Pygame on exit
pygame.quit()

```







```python
import os
import glob
import time
import pygame
from pygame.locals import *

# Initialize Pygame
pygame.init()

# Set up display (adjust resolution to match your display)
screen = pygame.display.set_mode((1024, 600), pygame.FULLSCREEN)

# Set a font and size
font = pygame.font.Font(None, 74)

# Initialize temperature sensor
os.system('modprobe w1-gpio')
os.system('modprobe w1-therm')

base_dir = '/sys/bus/w1/devices/'
device_folder = glob.glob(base_dir + '28*')[0]
device_file = device_folder + '/w1_slave'

# Function to read raw temperature data from the sensor
def read_temp_raw():
    with open(device_file, 'r') as f:
        lines = f.readlines()
    return lines

# Function to extract the temperature in Celsius and Fahrenheit
def read_temp():
    lines = read_temp_raw()
    while lines[0].strip()[-3:] != 'YES':
        time.sleep(0.2)
        lines = read_temp_raw()
    equals_pos = lines[1].find('t=')
    if equals_pos != -1:
        temp_string = lines[1][equals_pos+2:]
        temp_c = float(temp_string) / 1000.0
        temp_f = temp_c * 9.0 / 5.0 + 32.0
        return temp_c, temp_f
    return None, None

# Main loop to display temperature continuously
running = True
while running:
    # Event handling to exit on ESC key press
    for event in pygame.event.get():
        if event.type == QUIT:
            running = False
        elif event.type == KEYDOWN:
            if event.key == K_ESCAPE:  # Exit on pressing ESC
                running = False

    # Read temperature
    temperature_celsius, temperature_fahrenheit = read_temp()

    # Fill the screen with a color (black)
    screen.fill((0, 0, 0))

    # Create the text surfaces for Celsius and Fahrenheit
    temp_c_text = font.render(f'Temperature: {temperature_celsius:.2f} C', True, (255, 255, 255))
    temp_f_text = font.render(f'Temperature: {temperature_fahrenheit:.2f} F', True, (255, 255, 255))

    # Display the text on the screen at specified positions
    screen.blit(temp_c_text, (100, 200))  # Adjust coordinates as needed
    screen.blit(temp_f_text, (100, 300))  # Adjust coordinates as needed

    # Update the display
    pygame.display.flip()

    # Delay to control the update rate (e.g., 1 second between readings)
    time.sleep(1)

# Quit Pygame on exit
pygame.quit()

```

# **Running Script on Start (Method 1)**
Open the `.bashrc` file:
```shell
nano ~/.bashrc

```

At the end of the file, add the following line:
```shell
python3 /home/pi/display_temperature.py &

```


# *Running Script on Start (Method 2)*


Edit `rc.local`:

```shell
sudo nano /etc/rc.local

```

**Add the Python script command** before the line `exit 0`. Replace `/path/to/your_script.py` with the path to your Python script:
```shell
/usr/bin/python3 /path/to/your_script.py &
exit 0

```

- Here, `/usr/bin/python3` specifies the path to Python 3.
- The `&` symbol will run it in the background.

**Save and exit** (`Ctrl + X`, then `Y`, and `Enter`).

**Make sure the script is executable**:

```shell
chmod +x /path/to/your_script.py

```

Reboot to test:

```shell
sudo reboot

```



# **Display temperature Home UI**


![[PhotoScale UI.png]]

```python
# -*- coding: utf-8 -*-
import os
import glob
import time
import pygame
from pygame.locals import *

# Initialize Pygame
pygame.init()

# Set up display (adjust resolution to match your display)
screen = pygame.display.set_mode((1024, 600), pygame.FULLSCREEN)

# Set up fonts for title and temperature
title_font = pygame.font.Font(None, 120)  # Larger font for the title
font = pygame.font.Font(None, 150)        # Font for temperature

# Define colors (using RGB values)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
BLUE = (0, 0, 255)
TITLE_COLOR = (255, 215, 0)  # Gold color for title

# Initialize temperature sensor
os.system('modprobe w1-gpio')
os.system('modprobe w1-therm')

base_dir = '/sys/bus/w1/devices/'
device_folder = glob.glob(base_dir + '28*')[0]
device_file = device_folder + '/w1_slave'

# Function to read raw temperature data from the sensor
def read_temp_raw():
    with open(device_file, 'r') as f:
        lines = f.readlines()
    return lines

# Function to extract the temperature in Celsius and Fahrenheit
def read_temp():
    lines = read_temp_raw()
    while lines[0].strip()[-3:] != 'YES':
        time.sleep(0.2)
        lines = read_temp_raw()
    equals_pos = lines[1].find('t=')
    if equals_pos != -1:
        temp_string = lines[1][equals_pos+2:]
        temp_c = float(temp_string) / 1000.0
        temp_f = temp_c * 9.0 / 5.0 + 32.0
        return temp_c, temp_f
    return None, None

# Load and scale the icon
try:
    # Load an icon image if you have one (example path given)
    icon = pygame.image.load('/home/PhotoRX/Downloads/Tempicon.png')  # Replace with actual icon path
    icon = pygame.transform.scale(icon, (100, 250))  # Smaller dimensions for better fit
except:
    # Fallback: use a degree symbol rendered as text if the image is not found
    icon = font.render("\u00B0", True, WHITE)

# Main loop to display temperature continuously
running = True
while running:
    # Event handling to exit on ESC key press
    for event in pygame.event.get():
        if event.type == QUIT:
            running = False
        elif event.type == KEYDOWN:
            if event.key == K_ESCAPE:  # Exit on pressing ESC
                running = False

    # Read temperature
    temperature_celsius, temperature_fahrenheit = read_temp()

    # Fill the screen with a color (black)
    screen.fill((0, 0, 0))

    # Render and display the title centered
    title_text = title_font.render("PhotoScale", True, TITLE_COLOR)
    title_rect = title_text.get_rect(center=(screen.get_width() // 2, 60))  # Adjust vertical position
    screen.blit(title_text, title_rect)

    # Display the scaled temperature icon to the right of temperature values
    icon_x = 580  # Updated X-coordinate for positioning to the right of temperature
    icon_y = 150  # Updated Y-coordinate for centering vertically with text
    screen.blit(icon, (icon_x, icon_y))  # Position the icon in the desired area

    # Create the text surfaces with custom colors
    temp_c_text = font.render(f'{temperature_celsius:.2f} \u00B0C', True, RED)  # Red for Celsius
    temp_f_text = font.render(f'{temperature_fahrenheit:.2f} \u00B0F', True, BLUE)  # Blue for Fahrenheit

    # Display the temperature texts on the screen at specified positions
    screen.blit(temp_c_text, (100, 150))  # Adjust coordinates as needed
    screen.blit(temp_f_text, (100, 300))  # Adjust coordinates as needed

    # Update the display
    pygame.display.flip()

    # Delay to control the update rate (e.g., 1 second between readings)
    time.sleep(1)

# Quit Pygame on exit
pygame.quit()

```



# **Added Button and Settings Page**


![[prx.png]]
```python
# -*- coding: utf-8 -*-
import os
import glob
import time
import pygame
from pygame.locals import *

# Initialize Pygame
pygame.init()

# Set up display (adjust resolution to match your display)
screen = pygame.display.set_mode((1024, 600), pygame.FULLSCREEN)

# Set up fonts for title, temperature, and smaller side buttons
title_font = pygame.font.Font(None, 120)  # Font for the title
temp_font = pygame.font.Font(None, 150)   # Font for temperature
button_font = pygame.font.Font(None, 30)  # Smaller font for side buttons
settings_font = pygame.font.Font(None, 60)  # Smaller font for Settings screen text

# Define colors (using RGB values)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
BLUE = (0, 0, 255)
TITLE_COLOR = (255, 215, 0)  # Gold color for title
BUTTON_COLOR = (0, 128, 128)  # Base color for side buttons
BUTTON_HIGHLIGHT = (0, 180, 180)  # Highlight color for buttons on touch

# Initialize temperature sensor
os.system('modprobe w1-gpio')
os.system('modprobe w1-therm')

base_dir = '/sys/bus/w1/devices/'
device_folder = glob.glob(base_dir + '28*')[0]
device_file = device_folder + '/w1_slave'

# Function to read raw temperature data from the sensor
def read_temp_raw():
    with open(device_file, 'r') as f:
        lines = f.readlines()
    return lines

# Function to extract the temperature in Celsius and Fahrenheit
def read_temp():
    lines = read_temp_raw()
    while lines[0].strip()[-3:] != 'YES':
        time.sleep(0.2)
        lines = read_temp_raw()
    equals_pos = lines[1].find('t=')
    if equals_pos != -1:
        temp_string = lines[1][equals_pos+2:]
        temp_c = float(temp_string) / 1000.0
        temp_f = temp_c * 9.0 / 5.0 + 32.0
        return temp_c, temp_f
    return None, None

# Define button class
class Button:
    def __init__(self, text, x, y, width, height, action):
        self.text = text
        self.rect = pygame.Rect(x, y, width, height)
        self.action = action
        self.color = BUTTON_COLOR

    def draw(self, screen):
        pygame.draw.rect(screen, self.color, self.rect, border_radius=10)
        label = button_font.render(self.text, True, WHITE)
        label_rect = label.get_rect(center=self.rect.center)
        screen.blit(label, label_rect)

    def is_clicked(self, pos):
        return self.rect.collidepoint(pos)

    def highlight(self):
        self.color = BUTTON_HIGHLIGHT

    def reset_color(self):
        self.color = BUTTON_COLOR

# Define actions for each button
def show_temperature_screen():
    global current_screen
    current_screen = "temperature"

def show_settings_screen():
    global current_screen
    current_screen = "settings"

# Initialize smaller buttons
buttons = [
    Button("Home", 20, 100, 100, 50, show_temperature_screen),
    Button("Settings", 20, 180, 100, 50, show_settings_screen),
]

# Load and scale the icon
try:
    icon = pygame.image.load('/home/PhotoRX/Downloads/Tempicon.png')  # Replace with actual icon path
    icon = pygame.transform.scale(icon, (100, 220))  # Smaller dimensions for better alignment
except:
    icon = temp_font.render("\u00B0", True, WHITE)  # Fallback: use degree symbol if image is missing

# Main loop to display screens
current_screen = "temperature"  # Start with the temperature screen
running = True
while running:
    for event in pygame.event.get():
        if event.type == QUIT:
            running = False
        elif event.type == KEYDOWN:
            if event.key == K_ESCAPE:
                running = False
        elif event.type == MOUSEBUTTONDOWN:
            pos = pygame.mouse.get_pos()
            for button in buttons:
                if button.is_clicked(pos):
                    button.highlight()  # Highlight button when clicked
                    button.action()  # Perform the button's action

        elif event.type == MOUSEBUTTONUP:
            for button in buttons:
                button.reset_color()  # Reset button color after release

    # Fill the screen with a color (black)
    screen.fill((0, 0, 0))

    # Draw side buttons on the left side
    for button in buttons:
        button.draw(screen)

    # Display the current screen based on current_screen variable
    if current_screen == "temperature":
        # Temperature screen
        title_text = title_font.render("PhotoScale", True, TITLE_COLOR)
        title_rect = title_text.get_rect(center=(screen.get_width() // 2, 60))
        screen.blit(title_text, title_rect)

        # Display temperature data
        temperature_celsius, temperature_fahrenheit = read_temp()
        temp_c_text = temp_font.render(f'{temperature_celsius:.2f} \u00B0C', True, RED)
        temp_f_text = temp_font.render(f'{temperature_fahrenheit:.2f} \u00B0F', True, BLUE)
        screen.blit(temp_c_text, (180, 130))
        screen.blit(temp_f_text, (180, 250))

        # Display the temperature icon next to the Celsius text
        icon_x = 620  # Position closer to the Celsius text
        icon_y = 130  # Adjusted Y-coordinate for alignment
        screen.blit(icon, (icon_x, icon_y))

    elif current_screen == "settings":
         # Settings screen
        settings_title = title_font.render("Settings", True, TITLE_COLOR)
        settings_rect = settings_title.get_rect(center=(screen.get_width() // 2, 60))
        screen.blit(settings_title, settings_rect)

        # Placeholder settings information with reduced font size
        setting_text_1 = settings_font.render("Right Reactor: Intensity", True, WHITE)
        setting_text_2 = settings_font.render("Left Reactor: Intensity", True, WHITE)
        screen.blit(setting_text_1, (200, 200))
        screen.blit(setting_text_2, (200, 300))

    # Update the display
    pygame.display.flip()
    time.sleep(0.1)  # Adjust refresh rate

# Quit Pygame on exit
pygame.quit()

```



# **Auto Run Script After Boot**

There are several ways to do this stuffs. We tried to use *systemd, supervisor, crontab* etc.
We were facing some errors like `XDG_RUNTIME_DIR` error often occurs when trying to run a GUI-based application (like a Pygame script) from a `systemd` service because `systemd` services do not typically have access to the graphical display environment that is set up for user sessions.



```bash
chmod +x /home/PhotoRX/TempUI.py  

```

```bash
sudo nano /etc/systemd/system/tempUI.service

```

Edited service file (`/etc/systemd/system/temp_monitor.service`) to set the `XDG_RUNTIME_DIR` and `DISPLAY` environment variables.
- **`DISPLAY=:0`** specifies the default display for the X server.
- **`XDG_RUNTIME_DIR=/run/user/1000`** is typically the path for the user session runtime directory (for user `pi`, with UID `1000`).

```ini
[Unit]
Description=Temperature Monitoring Script
After=multi-user.target

[Service]
Environment="DISPLAY=:0"
Environment="XDG_RUNTIME_DIR=/run/user/1000"
ExecStart=/usr/bin/python3 /home/pi/tempUI.py  # Replace with the correct path
WorkingDirectory=/home/PhotoRX
StandardOutput=inherit
StandardError=inherit
Restart=always
User=PhotoRX

[Install]
WantedBy=multi-user.target

```

**Enable and Start the Service**

```bash
sudo systemctl daemon-reload
sudo systemctl enable tempUI.service
sudo systemctl start tempUI.service

```

**Check the Service Status**

```bash
sudo systemctl status tempUI.service

```

use `journalctl` to view the logs:
```bash
journalctl -u tempUI.service

```


**Debugging**

```bash
sudo systemctl daemon-reload

```

```bash
sudo systemctl restart tempUI.service
sudo systemctl status tempUI.service

```




