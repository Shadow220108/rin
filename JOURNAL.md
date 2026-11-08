<div align="center">
  <h1>Rin</h1>
  <h3>by shadow</h3>
  <h3>total time spent : 22 hours </h3>
</div>

--- 

### journal #1 research and picking the chip 
time spent : 2 hours   

  
ok so every time i want to talk to a microcontroller over serial i end up borrowing someones usb to uart thing or using the sketchy one from the lab that only works half the time. so i thought why not just make my own, how hard can it be right (it was harder than i thought lol)  
  
first thing was picking the chip. i saw a lot of people use the ch340 or the cp2102 for these, but i ended up going with the mcp2200 from microchip. reasons:
- it does usb 2.0 to uart which is literally the whole point
- it has 8 gpio pins so i can break out extra pins and use them for other stuff later
- it has its own pins for tx and rx leds, so the blinky lights work without me doing anything extra
- the datasheet has a full example circuit which helped a LOT since this is my first usb chip board  

spent most of these 2 hours just reading the datasheet. like actually reading it, not skimming. figured out it needs a 12mhz crystal, which caps go where, what goes on the vusb pin and how the usb lines connect. also wrote down what i want on the board:
- some kind of usb port to plug into the laptop
- a way to switch between 3.3v and 5v logic
- tx, rx and power leds
- all the uart pins + gpios on headers so it fits on a breadboard

--- 

### journal #2 starting the schematic!  
time spent : 1.8 hours   

  
opened kicad and started with the main chip. the mcp2200 comes in a soic-20 package which is nice cuz its big enough that i could even hand solder it if i need to  
  
stuff i added around it:
- 12mhz crystal with two 22pF caps going to ground (C3 and C5)
- 4.7k pullup on the reset pin so it doesnt randomly reset
- 2.2uF and 100nF caps on vcc for decoupling
- 330nF cap on the vusb pin, the datasheet says this one is needed or the usb part acts weird  

i used net labels for basically everything instead of drawing wires across the whole page, because last time i did wires everywhere it turned into spaghetti and i couldnt find anything lol
  

<div align="center">
  
| mcp2200 schematic |
| --- |
| <img width="900" alt="mcp2200 schematic" src="images/mcp2200_schematic.png" /> |

</div>

it looks kinda empty here but everything is connected through the labels i promise

--- 

### journal #3 power and the voltage jumper  
time spent : 1.2 hours   

  
next up was power. usb gives 5v, but a lot of boards like the esp32 run on 3.3v and if you send 5v into their rx pin you can fry them. so i wanted to be able to choose  
  
added an rt9166 which is a 3.3v ldo regulator, it takes the 5v from usb and gives 3.3v. then i made a 3 pin header (P3) with VUSB on one end, +5VUSB on the other and VCC in the middle. put the jumper on one side and the board runs at 3.3v, put it on the other side and it runs at 5v. pretty simple but i was lowkey proud of this one
  

<div align="center">
  
| regulator and jumper |
| --- |
| <img width="900" alt="regulator and jumper" src="images/regulator_and_jumper.png" /> |

</div>

--- 

### journal #4 leds and breakout headers  
time spent : 1 hour   

  
for the headers i went with two 6 pin headers so the board can sit on a breadboard. took a bit to decide what goes where  
- P1 : GP1, RX, GP3, GP4, GP5, GND
- P2 : GP2, RTS, TX, CTS, GP0, GND  

so you get tx, rx, rts and cts for the uart stuff, and all the gpios that are free, plus ground on both sides
  

<div align="center">
  
| breakout headers |
| --- |
| <img width="700" alt="breakout headers" src="images/breakout_headers.png" /> |

</div>

then the leds! red on the TXLED pin and green on the RXLED pin so they blink when data is moving (i love blinky lights its the best part of any board), and a blue one for power so you know its plugged in. all three have 470 ohm resistors so they dont blind you

<div align="center">
  
| status leds |
| --- |
| <img width="400" alt="status leds" src="images/status_leds.png" /> |

</div>

--- 

### journal #5 usb input (mini b to usb c)  
time spent : 1.5 hours   

  
at first i drew the usb part with a mini b connector because thats what the symbol i found was. added a ferrite bead and a 4.7uF cap on vbus to filter out noise coming from the laptop  
  

<div align="center">
  
| first version, usb mini b |
| --- |
| <img width="800" alt="usb mini b" src="images/usb_mini_b_first.png" /> |

</div>

then i looked at it and was like bro who even has mini b cables anymore. so i switched to usb c, i used the HRO TYPE-C-31-M-12 since its cheap and jlc has it  
  
usb c was more work than i expected tho. you need a 5.1k resistor from CC1 to ground and another from CC2 to ground, if you skip these a c to c cable just wont give the board any power at all. i read about people making that exact mistake and having dead boards so i made sure to add them (R5 and R6). also usb c has two D+ and two D- pins (one set for each side of the plug) so i tied them together so it works whichever way you plug it in
  

<div align="center">
  
| usb c input |
| --- |
| <img width="900" alt="usb c input" src="images/usb_c_input.png" /> |

</div>

--- 

### journal #6 footprints and placing everything  
time spent : 2 hours   

  
schematic done (for now) so i moved to the pcb editor. picked footprints for everything, 0603 for all the resistors, caps and leds since thats small but still doable, soic-20 for the chip, sot-89 for the regulator and a small 3225 crystal  
  
made the board 40 x 30 mm with rounded corners. then the placing:
- usb c on the left edge so the cable comes out the side
- chip right in the middle
- headers along the top and bottom edges, 2.54mm apart so its breadboard friendly
- leds on the right side where you can actually see them  

the crystal and its caps went as close to the chip as i could get them, same with the decoupling caps, because the datasheet says long traces there can cause problems. moved stuff around for ages till the ratsnest stopped looking like a spiderweb
  

<div align="center">
  
| footprints placed |
| --- |
| <img width="800" alt="footprints placed" src="images/footprints_placed.png" /> |

</div>

--- 

### journal #7 routing (part 1)  
time spent : 3 hours   

  
the routing took the longest ngl. i started with the usb data lines (D+ and D-) since they should be short and run next to each other, then the crystal, then power, then everything else  
  
around the decoupling caps and the oscillator pins it was super tight, i had to zoom in like crazy to fit the traces between the pads. when a trace wouldnt fit on top i dropped a via and went on the back layer
  

<div align="center">
  
| zoomed in near the caps |
| --- |
| <img width="800" alt="routing closeup" src="images/routing_closeup.png" /> |

</div>

this is around halfway. the header side was so annoying because the gpio pins on the chip are in a totally different order than the headers, so the traces kept crossing each other. had to rip up and redo a few of them like 3 times

<div align="center">
  
| halfway there |
| --- |
| <img width="850" alt="routing halfway" src="images/routing_halfway.png" /> |

</div>

--- 

### journal #8 cleaning up the schematic  
time spent : 1.2 hours   

  
went back to the schematic because it was getting really messy, random parts everywhere and you couldnt tell what belonged to what. so i put each section in its own box with a title:
- mcp2200 usb-uart
- usb input
- 3.3v regulator + vcc select
- status leds
- breakout headers
- power flags  

way easier to read now, and if anyone wants to look at it later they can actually understand it. also helps me find stuff faster when i need to change something
  

<div align="center">
  
| full schematic |
| --- |
| <img width="900" alt="full schematic" src="images/full_schematic_cleaned.png" /> |
| <img width="900" alt="mcp2200 section" src="images/mcp2200_section_cleaned.png" /> |

</div>

--- 

### journal #9 fixing errors  
time spent : 2 hours   

  
ran erc and got hit with a whole wall of errors. most of them said "input power pin not driven by any output power pins". this is because kicad doesnt know that the power is coming in from the usb connector, so it thinks nothing is powering the chip. the fix is adding PWR_FLAG symbols on the power nets, so i made a little box just for those
  

<div align="center">
  
| power flags |
| --- |
| <img width="800" alt="power flags" src="images/power_flags_erc.png" /> |

</div>

also put no connect flags (the little x) on the usb c pins i dont use, SBU1 and SBU2, so erc stops yelling about those too  
  
then i updated the pcb from the schematic and ran drc. got some clearance errors where traces were too close to pads, and a bunch of silkscreen text sitting right on top of pads. fixed them one by one, rerouted the ones that were too close, moved the text around. kept running drc again and again until it was finally happy, felt so good

--- 

### journal #10 finishing the routing and ground pour  
time spent : 1.8 hours   

  
finished routing everything else after the fixes. then added a ground pour on both the top and the bottom layer, so all the ground pads connect through the copper fill and i didnt have to route ground to every single part by hand. added extra vias around the board to stitch the top and bottom ground together, ended up with 29 vias total  
  

<div align="center">
  
| fully routed |
| --- |
| <img width="850" alt="routed pcb" src="images/pcb_routed.png" /> |

| with the ground pour |
| --- |
| <img width="850" alt="ground pour" src="images/ground_pour.png" /> |

</div>

it looks so much more like a real board with the pour on :D

--- 

### journal #11 3d view check  
time spent : 0.7 hours   

  
opened the 3d viewer to check if everything actually fits. made sure the usb c is right at the edge so the cable can go in, nothing is overlapping and the headers are lined up. the header models have the pins sticking out of the bottom which looks funny but thats how they'll look after soldering anyway
  

<div align="center">
  
| first 3d look |
| --- |
| <img width="800" alt="3d first look" src="images/3d_first_look.png" /> |
| <img width="800" alt="3d top view" src="images/3d_top_view.png" /> |

</div>

it looked clean but kinda boring. needed some art :D

--- 

### journal #12 art time!  
time spent : 2.5 hours   

  
my fav part of making any pcb!! first i had to name it, i went with Rin because it sounds cute and short. made a cursive "Rin" logo for the front and a "made with love" text for the back, then imported both with kicads image converter so they turn into silkscreen
  

<div align="center">
  
| rin logo | made with love |
| --- | --- |
| <img width="323" alt="rin logo" src="images/rin_logo.png" /> | <img width="500" alt="made with love" src="images/made_with_love.png" /> |

</div>

the rin logo went right next to the chip, and i added a small hack club flag in the corner. on the back i wrote "made with love by shadow" across the whole thing. also hid all the component values on the silkscreen since they were overlapping everything and kept only the names like R1 and C2, it looks way cleaner now  
  
took a while to get the sizes right so the text doesnt run into any pads or vias, had to scale and move it a bunch of times

<div align="center">
  
| front | back |
| --- | --- |
| <img width="500" alt="front art" src="images/front_art.png" /> | <img width="500" alt="back art" src="images/back_art.png" /> |

</div>

--- 

### journal #13 final renders, gerbers and github  
time spent : 1.3 hours   

  
almost done! tried the gold pad look in the 3d viewer and it looks sooo good with the black board, took the final renders with that  
  
then made the gerbers, drill files, bom and the position file for jlcpcb. got a quote for 5 pcbs and assembly for 2 of them, it came to about 34$ with shipping, which is not bad for my own usb to uart board. after that i wrote the readme, sorted all the files into folders and pushed everything to github
  

<div align="center">
  
| final front | final back |
| --- | --- |
| <img width="500" alt="final front" src="images/final_front_gold.png" /> | <img width="500" alt="final back" src="images/final_back_gold.png" /> |

</div>

this would be the last journal, if you read till here, thank you :D
