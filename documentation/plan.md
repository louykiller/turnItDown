I want to build a mobile game for both Android and IOS where the objective of the game is to use a lot of the phone's sensors and different inputs do complete mini levels to raise awareness for the amount of information that the mobile apps have of our device.

I want a simple cartoonish look and feel to the app, nothing to complicated and I want to use a framework that allows me to access all the following inputs easily.

This app will use the following device's inputs/settings/events:
- display brightness
- volume percentage
- gyroscope
- proximity sensor
- microphone
- screenshot event
- ambient light sensor
- battery and charging status
- airplane mode
- system time/date
- flashlight
- barometer
- touch size and pressure
- compass
- multi-touch
- face expression recognition through front camera
- bluetooth connections
- light/dark mode
- power saving mode
- location
- clipboard
- text size
- step counter
- screen lock
- biometrics
- Wi-Fi/mobile data

For the start of the game the player needs to make a few things:
- Accept all permissions
- Put the device in light mode
- Put the volume at 50%
- Put the device display brightness at 50%
- Make sure airplane mode is off
- Make sure that the date/time is updated
- Have no Bluetooth connections
- Make sure the device is not in power saving mode
- Be in a well lit room 
- Do not have the phone charging
- Location off

After each level is passed using a certain sensor or setting (like brightness, volume, charging, airplane mode, flashlight, light/dark mode, power saving mode, bluetooth connections) the user must revert the action before the next level. For this it would be good to keep an initial state before each level of the different settings and sensors to alert the user what to change back. Make reverting part of the story.

I already planned a few levels:
1. A baby is trying to sleep in a crib but the ceiling light is to bright. The baby will keep crying until the player reduces the display brightness to its minimum, turning off the light in the game. This will use the device's display brightness.

2. A speaker is playing music loudly. The user needs to physically click the lower volume button on the device to turn off the speaker. This will use the device's volume percentage.

3. Water is running out of the tap and flooding the sink. The user needs to close the tap by rotating the phone 360 degrees on the horizontal plane. Make the in-game handle turn with the phone, and require clockwise. This will use the device's gyroscope.

4. A old landline phone is outside its base and is ringing. The user must put it back on the base and make it stop ringing by turning the phone screen down on a table. This will use the device's proximity sensor and accelerometer. where proximity only works in calls, accept "face down" alone.

5. A dog is barking loudly. The user must yell something to shut the dog up. This will use the device's microphone and a certain threshold must be met. If the user uses the volume button print a message teasing the user like "Just because you can't hear the dog it doesn't mean it stopped barking". The microphone will hear the game's own barking. Use echo cancellation or leave silent gaps between barks.

6. A ghost hunter is in a haunted living room with an old "spirit camera". The ghosts are invisible: you only see a chair rocking, a book floating, a candle flickering. The user must take a screenshot at the right moment to photograph the ghosts. Each screenshot shows a developed photo with any ghosts that were in the room at that moment. The user must capture all 3 ghosts using no more than 6 photos. This will use the device's screenshot event listener.

7. A TV is on and the user needs to point the remote at it to turn it off. At the beggining of the level the app should use the phone orientation as its base level where the remote is pointing at the ceiling and the user must point their phone down about 90 degrees to "point" to the TV. This will use the device's accelerometer.

8. There is a car dashboard with the wheel and right in front of the window the sun is blasting light. The user must cover the top of the device with their hand to block the ambient light sensor and make the character in game put its hand in front of the sun reducing the light being blasted. This will use the device's ambient light sensor or front camera.

9. A person is connected with cables to a hospital machine but the machine is unplugged and is slowly loosing battery. The user must charge their device to plug the machine back. This will use the device's battery charging status.

10. A flight attendant is yapping non-stop for the plane lift-off. The user must put their device in airplane mode. This will use the device's airplane mode.

11. A character enters a crime scene just after the criminal shoot someone. The user must go back in time a few minutes so that the character can time travel and stop the murder. This will use the device's system date/time. Make it so that it should only accept up to 10 minutes before, if it goes back further no character is at the scene (because it didn't happen yet) and if goes forward in time show a pool of blood after the body was removed or some police signs. Catch time-zone changes and tease them ("that's jet lag, not time travel").

12. A prisoner is escaping prison but the officers cannot see him in the dark. The user must turn on their flashlight to turn on a spotlight in game to spot the prisoner. This will use the device's flashlight.

13. A teacher is checking the attendance and the user name is called. The user needs to physically stand up and raise their phone up to show they are present. This will use the device's barometer. This is supposed to pick up the altitude change with the air pressure change. Standing up changes air pressure by only about 0.1 hPa, close to the sensor's noise, with a 1–2 second delay. Needs testing

14. An elbow is bleeding after a bike fall. The user must apply pressure with their finger, ideally with their thumb to get the bigger radius of "pressure" on the screen. This will use the device's Touch Size and pressure. Have the player tap with their index finger first, compare the thumb to that, and require holding for about 3 seconds.

15. A man is trying to put up a painting while a women in nagging him. The user must level the painting by physically tilt their device. This will use the device's accelerometer.

16. A fly is buzzing and flying around the player and it can go off screen. The user must physically rotate their phone to see where the fly went and tap the screen to kill it. This will use the device's compass combined with the gyroscope. the raw compass jitters so there needs to be some compensation

17. A big mouth is yapping and spitting while talking. The user must close the mouth by pinching with all 5 fingers on the screen. This will use the device's multi-touch detection.

18. A kid is mocking the player by sticking his tongue out. The user must also stick their tongue out. This will use the device's camera and expression recognition.

19. A group of people is talking loudly on the bus/metro and the player has a headset in sight. The user should connect to a headset by bluetooth. This will use the device's bluetooth audio conections.

20. A rooster is crowing loudly on a fence and the sun is shinning in the sky. The user must switch to dark mode in their device making the sun go down and the moon appearing and shushing the rooster. This will use the device's light/dark mode.

21. A 10 outlet power strip has 10 devices connected to it and it's pulling a lot of power making sparks and light to come off. The user must turn on power saving mode on. This will use the device's power saving mode.

22. A food delivery is trying to check your location but he can't find you. The user must turn location on. This will use the device's location.

23. The player needs to enter a super long and complicated password to turn off the alarm of the house. The user has nowhere to nput the password so they must copy the password. After it was copied it will access the clipboard and pass the level. This will use the device's clipboard.

24. The player is in a restaurant with a menu with super small letters and a waiter is pressuring the user to order food. The player must type the dish name exactly as it is in the menu. In order to read it properly the user must update the text size to large. This will use the device's text size.

25. A medic is telling the player they are 20 steps short for their daily goal. The user must walk 20 steps. This will use the device's step counter.

26. A burguer comes to the table and the player must add ketchup to it but it is stuck and won't come out. The user must shake the phone. This will use the device's accelerometer.

27. The player is lying on the bed late at night still gaming on the tablet at 3 a.m. and their mother walks in the room. The user must lock the tablet by locking the screen of the device itself. This will use the device's screen lock.

28. The player must enter a PIN to enter a bank app but of course they don't know the pin. There will be an option to use biometrics (Fingerprint / Face ID). They need to use it successfully to open. This will use the device's biometrics. It also teaches that the app only gets "yes/no", never your fingerprint.

29. There is a computer playing a youtube video which is buffering a lot and with terrible quality. The user must switch to mobile data. This will use the device's wifi and mobile data.

Note: "user" is the person playing the game. "character"/"player" is an in game simulation of the user's perspective. 


Other points:
- Show what was learned after each level and some trivia about the setting/input used, e.g. "Battery: 47%, no permission needed", and finish with a "here's what this game knows about you" report. Keep everything on the phone with no analytics.
- Give every level a skip and a "not supported on this device" message.
- Let the user explore to solve the level and progressively give more tips. The objective is that each level can be made in under 2 minutes. If they are stuck after 2 minutes add a (?) sign where the user can click and it shows the solution
