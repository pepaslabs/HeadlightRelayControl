# HeadlightRelayControl

A circuit board to control automotive headlight [relays](http://en.wikipedia.org/wiki/Relay).

![](releases/v1/top.png)

## Designing this circuit in LTSpice

Let's design this circuit using [LTSpice](http://www.linear.com/ltspice).

Start with the following basic circuit.  The [inductor](http://en.wikipedia.org/wiki/Inductor) L1 represents the coil inside of the relay.

![](github%20media/Clipboard02.png)

### Specifying the relay coil parameters

Before we can simulate this circuit, we need to know some characteristics of the relay coil.  Specifically, we need to know the [inducatance](http://en.wikipedia.org/wiki/Inductance) and the [resistance](http://en.wikipedia.org/wiki/Electrical_resistance_and_conductance).

Here I've measured the inductance of a typical 12 Volt, 30 Amp automotive relay to be about 132 millihenries (using a cheap [LC100-A](http://www.ebay.com/sch/i.html?_nkw=LC100-A) meter from eBay):

![](github%20media/Photo%20Mar%2020%2C%208%2008%2051%20PM.jpg)

Here, I've pulled up the [datasheet](http://www.te.com/commerce/DocumentDelivery/DDEController?Action=srchrtrv&DocNm=1432785-1&DocType=Customer+Drawing&DocLang=English) of a similar relay [listed on digikey.com](http://www.digikey.com/product-detail/en/1432785-1/PB680-ND/807757), where the resistance is listed as being 90 Ohms:

![](github%20media/Clipboard03.png)

![](github%20media/Clipboard04.png)

Right-click on L1 and fill in these parameters:

![](github%20media/Clipboard05.png)

We want to simulate turning this relay on.  We can do this by creating a **PULSE** voltage source which will switch from 0V to 12V at 10 milliseconds.  Right-click on V1, then click advanced and fill in these parameters:

![](github%20media/Clipboard07.png)

Simulate the relay for 100 milliseconds:

![](github%20media/Clipboard10.png)

![](github%20media/Clipboard09.png)

Click on the green **I(L1)** label to create a cursor.  We can use the cursor to meausre the current, which is 133mA:

![](github%20media/Clipboard12.png)

### Optimization: PICK and HOLD current

If you've ever operated an automotive relay at full voltage for a long period of time, you know they can get pretty hot.

It turns out that isn't necessary.  A relay only needs a short burst of full voltage (actually, current) to get the armature moving (this is called the **PICK** current).  Once switched, it requires only a fraction of that current (typically less than half of **PICK**) to keep the relay on (the **HOLD** current).

#### Current-limiting resistor

We can add a 100 Ohm current-limiting resistor after the relay coil:

![](github%20media/Clipboard14.png)

Now we have our **HOLD** current (63mA), but we've lost our **PICK** current.  This means our relay might not turn on reliably.

How can we have both a **PICK** and a **HOLD** current?  There are a few ways to implement this optimization.

#### PWM the coil

We could [PWM](http://en.wikipedia.org/wiki/Pulse-width_modulation) the coil, perhaps using a purpose-built [IC](http://en.wikipedia.org/wiki/Integrated_circuit), such as the [DRV120](http://www.ti.com/lit/ds/symlink/drv120.pdf) made by [TI](http://www.ti.com/).

However, there even simpler, more hobbyist-friendly solutions...

#### Bypass the current-limiting resistor

The technique I decided to use was to bypass the current-limiting resistor with a capacitor.

![](github%20media/Clipboard25.png)

I came across this trick in a [blog post](http://jumperone.com/2011/10/using-relays/) by [Phil Levchenko](http://jumperone.com/about/).  Be sure to also check out his [YouTube channel](https://www.youtube.com/user/JumperOneTV).

First we will try a 10uF capacitor.  We will esitmate its [ESR](http://en.wikipedia.org/wiki/Equivalent_series_resistance) to be 1 Ohm.

![](github%20media/Clipboard18.png)

Also, shorten the simulation window to just 50 milliseconds:

![](github%20media/Clipboard17.png)

![](github%20media/Clipboard19.png)

It worked!  We have both a **PICK** and a **HOLD** current.

However, our **PICK** current is small and not very long in duration, so a 10uF capacitor might not be enough to give us reliable relay operation.

Based on the [datasheet](http://www.te.com/commerce/DocumentDelivery/DDEController?Action=srchrtrv&DocNm=1432785-1&DocType=Customer+Drawing&DocLang=English) mentioned above, we should shoot for a **PICK** current window of around 8 milliseconds in duration, and we want it to be much closer to 133mA.

Next we'll try a 100uF capacitor.  Estimate the ESR to be 100 milliohms.

![](github%20media/Clipboard16.png)

![](github%20media/Clipboard22.png)

That's a lot closer to what we want.  Use a cursor to get a more exact idea of what our 8ms **PICK** window looks like.

![](github%20media/Clipboard23.png)

Just to be thorough, we should also try a larger capacitor.  Try a 220uF (with 100mOhm ESR):

![](github%20media/Clipboard20.png)

![](github%20media/Clipboard26.png)

It looks like either 100uF or 200uF would work well.

When designing a circuit, you'll often find yourself in these "[Goldilocks](http://en.wikipedia.org/wiki/The_Story_of_the_Three_Bears)" scenarios, trying to find the value that's "just right".  Its a good idea to reduce the friction of that process, so that you can iterate on a design faster.

The quick-n-dirty way to do this is to triplicate your circuit and simulate all three at once.

![](github%20media/Clipboard24.png)

Eventually though, you'll want to learn how to [step the parameters of your simulations](http://www.linear.com/solutions/1089).

## Background: The need for this circuit

### Why did I make this circuit?

After upgrading the aging, dim headlights on my (1997 Dodge Ram) pickup truck with an HID headlight kit from eBay, I ran into a problem: the stock headlight switch couldn't handle the higher current load of the HID headlights.

The spade connector on the headlight switch got so hot that the plastic which holds it in place started to melt.  Some of the melted plastic interfered with the connection, and the connector was no longer being securely held in place.  The result was that you had to wiggle the switch to get the headlights to come, and you could occaisionally smell burning plastic.  Eventually, the connection stopped working entirely.

![](github%20media/Photo_Mar_18%2C_10_02_26_PM_032115_125033_PM.jpg)

![](github%20media/Photo_Mar_18%2C_10_04_28_PM_032115_125230_PM.jpg)

### Why did the stock headlight switch fail?  Its hard to beleive the HIDs drew THAT MUCH more current...

True, there's more to this story than a simple matter of higher current draw.

Stock halogen headlights are spec'ed at 55 Watts.  If we assume that spec is for 12 Volts (rather than the 14.2 Volts you'd find in a running car), that means each lamp is drawing 55 / 12 = roughly 4.6 Amps.

The HID ballasts I use claim a max current draw of 10 Amps (they don't always draw 10 Amps, but they can go that high).  That's potentially more than twice the current which the stock headlight switch and wiring were designed around.

#### HID ballasts look like a constant-wattage load

However, this problem is compounded by the fact that the HID ballasts are basically a constant-current supply, which means they look like a constant-wattage load to your car's electrical system.  This means that if you try to feed the ballast a lower voltage, it will simply draw more current to make up for it.  That's why they stay the same brightness even if your voltage droops a bit.

Resistive loads (like a traditional halogen bulb) don't behave like this.  If you lower the voltage, they draw fewer amps (and get dimmer).  If I had put 100 watt halogen bulbs in my truck, that would be bad (because they would try to draw 100 / 12 = 8.3 Amps), but it wouldn't be as precarious as the situation created by the HID ballasts.  Here's why:

When we turn on the 100 Watt halogen bulbs, they would draw 8.3 Amps.  The headlight switch (particularly the spade connector) and the wiring would start to heat up because we are drawing more current than they were rated for.  As they heat up, and their resistance would increase, which would create a voltage drop.  So now your bulbs only see maybe 10 Volts.  This means they draw less current, which means the headlight switch and wiring could start to cool down a bit, and eventually the whole system would settle out at some equilibrium.  This is how linear, resistive loads work.

#### Thermal runaway

HID ballasts don't work like that.  The ballasts say their max draw is 10 Amps, but let's assume their normal, 12 Volt draw is only 6 Amps (72 Watts).  What happens when we turn them on?  Initially, the headlight switch and wiring aren't hot, and their resistance is low, and the HID ballast sees pretty close to 12 Volts, and draws 6 Amps.  But even 6 Amps is more than what the stock switch and wiring was designed to handle, so they start heating up, increasing in resistance, and creating a Voltage drop.  The HID ballasts are now only seeing 10 volts, and they compensate by increasing their current draw to 7.2 Amps (still 72 Watts).  This increased current draw causes the switch and wiring to heat up even more, become more resistive, and create an even bigger drop.  The ballast now sees only 8 Volts, and responds by drawing 9 Amps (still 72 Watts).  And now the switch and wiring heat up even more, and so on, and so on...

This situation is referred to as "thermal runaway".  The process would keep going indefinitely, except that the HID ballasts are designed with a safety mechanism where they will refuse to draw more than 10 Amps.  But at that point, the headlight switch and wiring are already hot enough to start burning the plastic and damaging the switch.

#### Solution: use relays!

The solution to all of this is to use relays, which are high-current switches which are operated by a small current.  Typical automotive relays are rated to handle 30 Amps (more than enough for our HID ballasts) and they are typically controlled by about 0.125 Amps (125 milliamps).  This looks like a tiny, tiny load to your headlight switch, which means it won't even get warm, let alone hot enough to self-destruct.
