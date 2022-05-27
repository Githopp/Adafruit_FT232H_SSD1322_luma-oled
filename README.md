
# Using the Adafruit FT232H with luma-oled on SSD1322

## how to wire
<div align="center">
  <img src="wiring.jpg"><br>
</div>

from luma.core.render import canvas
from luma.core.interface import serial

serial_interface = serial.ftdi_spi(device='ftdi://ftdi:232h:1/1', bus_speed_hz=16000000,  gpio_CS=3, gpio_DC=4, gpio_RST=5,reset_hold_time=0, reset_release_time=0)

device = ssd1322(serial_interface, rotate=2)

#device = ssd1322(serial_interface = serial_interface, rotate=2, mode = "1", framebuffer = "diff_to_previous")
#device = ssd1322(serial_interface = serial_interface, rotate=2, framebuffer = "diff_to_previous")
#device.size


import math
import time
import datetime
from luma.core.render import canvas


def posn(angle, arm_length):
    dx = int(math.cos(math.radians(angle)) * arm_length)
    dy = int(math.sin(math.radians(angle)) * arm_length)
    return (dx, dy)


def main():
    today_last_time = "Unknown"
    while True:
        now = datetime.datetime.now()
        today_date = now.strftime("%d %b %y")
        today_time = now.strftime("%H:%M:%S")
        if today_time != today_last_time:
            today_last_time = today_time
            with canvas(device) as draw:
                now = datetime.datetime.now()
                today_date = now.strftime("%d %b %y")

                margin = 4

                cx = 30
                cy = min(device.height, 64) / 2

                left = cx - cy
                right = cx + cy

                hrs_angle = 270 + (30 * (now.hour + (now.minute / 60.0)))
                hrs = posn(hrs_angle, cy - margin - 7)

                min_angle = 270 + (6 * now.minute)
                mins = posn(min_angle, cy - margin - 2)

                sec_angle = 270 + (6 * now.second)
                secs = posn(sec_angle, cy - margin - 2)

                draw.ellipse((left + margin, margin, right - margin, min(device.height, 64) - margin), outline="white")
                draw.line((cx, cy, cx + hrs[0], cy + hrs[1]), fill="white")
                draw.line((cx, cy, cx + mins[0], cy + mins[1]), fill="white")
                draw.line((cx, cy, cx + secs[0], cy + secs[1]), fill="red")
                draw.ellipse((cx - 2, cy - 2, cx + 2, cy + 2), fill="white", outline="white")
                draw.text((2 * (cx + margin), cy - 8), today_date, fill="yellow")
                draw.text((2 * (cx + margin), cy), today_time, fill="yellow")

        time.sleep(0.1)


main()
