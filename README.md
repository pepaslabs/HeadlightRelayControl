# HeadlightRelayControl
A circuit board to control automotive headlight relays.

## How this circuit was designed

Let's design this circuit using LTSpice.

Start with the following basic circuit.  The inductor L1 represents the coil inside of the relay.

![](github%20media/Clipboard02.png)

### Specifying the relay coil parameters

Before we can simulate this circuit, we need to know some characteristics of the relay coil.  Specifically, we need to know the inducatance and the resistance.

Here I've measured the inductance of a typical 12 Volt, 30 Amp automotive relay to be about 132 millihenries (using a cheap [LC100-A](http://www.ebay.com/sch/i.html?_nkw=LC100-A) meter from eBay):

![](github%20media/Photo%20Mar%2020%2C%208%2008%2051%20PM.jpg)

Here, I've pulled up the [datasheet](http://www.te.com/commerce/DocumentDelivery/DDEController?Action=srchrtrv&DocNm=1432785-1&DocType=Customer+Drawing&DocLang=English) of a similar relay [listed on digikey.com](http://www.digikey.com/product-detail/en/1432785-1/PB680-ND/807757), where the resistance is listed as being 90 Ohms:

![](github%20media/Clipboard03.png)

![](github%20media/Clipboard04.png)

Right-click on L1 and fill in these parameters:



### Background: The need for this circuit

#### Why did I make this circuit?

After upgrading the aging, dim headlights on my (1997 Dodge Ram) pickup truck with an HID headlight kit from eBay, I ran into a problem: the stock headlight switch couldn't handle the higher current load of the HID headlights.

The spade connector on the headlight switch got so hot that the plastic which holds it in place started to melt.  Some of the melted plastic interfered with the connection, and the connector was no longer being securely held in place.  The result was that you had to wiggle the switch to get the headlights to come, and you could occaisionally smell burning plastic.  Eventually, the connection stopped working entirely.

![](github%20media/Photo_Mar_18%2C_10_02_26_PM_032115_125033_PM.jpg)

![](github%20media/Photo_Mar_18%2C_10_04_28_PM_032115_125230_PM.jpg)

#### Why did the stock headlight switch fail?  Its hard to beleive the HIDs drew THAT MUCH more current...

True, there's more to this story than a simple matter of higher current draw.

Stock halogen headlights are spec'ed at 55 Watts.  If we assume that spec is for 12 Volts (rather than the 14.2 Volts you'd find in a running car), that means each lamp is drawing 55 / 12 = roughly 4.6 Amps.

The HID ballasts I use claim a max current draw of 10 Amps (they don't always draw 10 Amps, but they can go that high).  That's potentially more than twice the current which the stock headlight switch and wiring were designed around.

##### HID ballasts look like a constant-wattage load

However, this problem is compounded by the fact that the HID ballasts are basically a constant-current supply, which means they look like a constant-wattage load to your car's electrical system.  This means that if you try to feed the ballast a lower voltage, it will simply draw more current to make up for it.  That's why they stay the same brightness even if your voltage droops a bit.

Resistive loads (like a traditional halogen bulb) don't behave like this.  If you lower the voltage, they draw fewer amps (and get dimmer).  If I had put 100 watt halogen bulbs in my truck, that would be bad (because they would try to draw 100 / 12 = 8.3 Amps), but it wouldn't be as precarious as the situation created by the HID ballasts.  Here's why:

When we turn on the 100 Watt halogen bulbs, they would draw 8.3 Amps.  The headlight switch (particularly the spade connector) and the wiring would start to heat up because we are drawing more current than they were rated for.  As they heat up, and their resistance would increase, which would create a voltage drop.  So now your bulbs only see maybe 10 Volts.  This means they draw less current, which means the headlight switch and wiring could start to cool down a bit, and eventually the whole system would settle out at some equilibrium.  This is how linear, resistive loads work.

##### Thermal runaway

HID ballasts don't work like that.  The ballasts say their max draw is 10 Amps, but let's assume their normal, 12 Volt draw is only 6 Amps (72 Watts).  What happens when we turn them on?  Initially, the headlight switch and wiring aren't hot, and their resistance is low, and the HID ballast sees pretty close to 12 Volts, and draws 6 Amps.  But even 6 Amps is more than what the stock switch and wiring was designed to handle, so they start heating up, increasing in resistance, and creating a Voltage drop.  The HID ballasts are now only seeing 10 volts, and they compensate by increasing their current draw to 7.2 Amps (still 72 Watts).  This increased current draw causes the switch and wiring to heat up even more, become more resistive, and create an even bigger drop.  The ballast now sees only 8 Volts, and responds by drawing 9 Amps (still 72 Watts).  And now the switch and wiring heat up even more, and so on, and so on...

This situation is referred to as "thermal runaway".  The process would keep going indefinitely, except that the HID ballasts are designed with a safety mechanism where they will refuse to draw more than 10 Amps.  But at that point, the headlight switch and wiring are already hot enough to start burning the plastic and damaging the switch.

##### Solution: use relays!

The solution to all of this is to use relays, which are high-current switches which are operated by a small current.  Typical automotive relays are rated to handle 30 Amps (more than enough for our HID ballasts) and they are typically controlled by about 0.125 Amps (125 milliamps).  This looks like a tiny, tiny load to your headlight switch, which means it won't even get warm, let alone hot enough to self-destruct.
