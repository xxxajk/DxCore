# Overall information regarding the AVR Dx-series
In early 2020, as the world was shutting down because of COVID-19, Microchip released the first of their new generation of parts: The AVR Dx-series.

## Big Picture
These parts depart from the naming scheme used for AVR devices in the past; these are named AVR followed by the size of the flash, in KB, followed by DA, DB, or DD (depending on the "series" or "family", then the number of pins. Note that the pin count also determines how many of certain peripherals the parts have available - parts with more pins have more peripherals to do things with those pins. 64-pin parts are not available in 32k flash size. The 128k flash size is the highest that can be supported with a 16-bit program counter (above that, a number of instructions become slower, and everything gets more complicated), and with the current scheme for interacting with the pins, the 55 I/O pins (56 less the UPDI pin which takes the place of PF7) are the limit of what a modern AVR can accommodate while allowing single cycle access to all pins - so these take them to the top end of what is possible without extensions to the architecture.

At present, it appears that there will be at least three lines of AVR Dx-series parts: The "baseline" DA, the DB with multivoltage (MVIO) and on-chip OPAMPs, and the "budget" DD-series which has MVIO but otherwise far fewer peripherals, and is available in pincounts as low as 14, bringing them dangerouly close to the tinyAVR parts. There is *potentially* a new "DU" series with DD-like low pincounts, which instead of `MVIO`, sacrifice most of `PORTC` in the name of **native USB**. A product brief was available, but only, as the name suggests, briefly. So why did they retract it, and why so quickly? Was the project canceled? Did they decide to radically change the design so it wasn't accurate? Were they just afraid that if they put up a product brief years before the product launched, it would hurt sales of their existing native USB AVRs, (the aging ATmega32u4 and it's smaller versions). I think it's fair to say that it is unlikely to have been due to inaccuracies in the product brief (barring a major change that would have left the product brief describing something totally not what the product was) - all of the recent product briefs have had at least a few small errors on them (most are clearly attributable to the creator of the product brief, rather than a change being made in the design). Hopefully they haven't canceled the native USB modern AVR thing altogether.  As long as there is a native USB modern AVR in the not-too-distant future, I'll be happy with any plausible configuration. The prospect of a new, more capable AVR with native USB is exciting indeed, since the 32u4 was getting a little long in the tooth. This core supports the DA, DB, and DD-series and pending availability, will support the EA-series unless the scale of changes there is greater than expected and a separate core is required (this does not appear to be the case) If/when The DU-series materializes, they would also fall under the perview of this core.

### DA-series
The "baseline" full-size parts - however much I was in awe of these when they were first released, having seen the DB-series, it now appears that these are more akin to a 0-series than a 1-series - in every arena, the DB has the same or better. They do not support using an external crystal for the main clock, like the other Dx parts do, but the internal oscillator on these parts is still WAY better than the classic AVRs had - all the ones I've tested are weithin half a percent at room temp and typical operating voltages, even without autotune... To make sure autotune was working, I had to point a torch at it, because I couldn't get enough of a change in the internal oscillator frequency from changing the supply voltage - in fact, it didn't seem to change at all. It is also the only currently announced Dx series without `MVIO`. While they may not shine as brightly next to the other Dx lines. The fact that these look less than stellar beside the DB doesn't change the fact that they absolutely bury any AVR from before 2020. There is only one thing that they have and the DB doesn't - a peripheral touch controller (which we can't use in Arduino land because they won't share the source code and force you to use their IDE.)

### DB-series
The DB-series is almost an exact copy of the DA-series (they fixed some of the most egregious silicon bugs, but only some), only with a few even more exciting features tacked on: Support for a real high-frequency crystal as clock source (finally - first time since the modern AVR architecture was released in 2016), "MVIO" (multivoltage I/O), where PORTC can run at a different voltage than the rest of the chip (essentially, a builtin bidirectional level shifter). The other "headline feature", is the  two (28/32-pin parts) or three (48/64-pin parts) on-chip opamps, with software-controlled MUX for the inputs and an on-chip feedback resistor ladder. These can be used as gain stage for the ADC, for example, or to buffer the DAC output (though these opamps can't supply **that** much current, they can supply tens of mA, instead of ~ 1 like the unaided DAC), and connected to each other for multistage amplifiers (on parts with 3, you can even connect them together as an instrumentation amplifier). The included `<Opamp.h>` library provides a minimalist wrapper around them exposing all of the functionality.

### DD-series
The DD-series is a smaller-pincount lower priced product line; parts are available with 14-32 pins. They've got the `MVIO` (3 or 4 MVIO pins depending on pincount), and the UPDI pin can be configured as an I/O pin (in this case, further UPDI programming requires an HV pulse to be applied to the *reset* pin, while the reset pin, like the DA and DB parts, can be configured only as reset or input. (This sidesteps the awkward problem that tinyAVR has of "How do I put an HV pulse on the reset pin when it's an output being driven low?") One thing worth noting is that the 28-pin and 32-pin DD-series parts are, but for expanded port multiplexing options for `SPI0`, `USART0`, and `TCD0` (the DA/DB were supposed to support TCD0 PORTMUX options, and a future die revision may fix that), and the addition of ADC input functionality on pins in `PORTA` and `PORTC`, inferior to the DB: it's a DB with fewer copies of peripherals and no opamps.

If you were hoping to use a tinyAVR breakout board for the 14 or 20-pin versions, I hate to be the bearer of bad news, but the tinyAVR boards won't work. They swapped power and ground on the SOIC packages vis-a-vis tinyAVR, and the layout of the VQFN20 is different than the 20-pin tinyAVRs. (The 28 and 32 pin parts are pin compatible with AVR-DB - but as I said, I don't think anyone is standing in line for those - except maybe folks who can't get their hands on the DA or DB parts). Actually, no, I don't hate being the bearer of this news. In fact, it brings me joy - I sell breakout boards, remember? And now everyone's going to need different ones than they'd been using for 14 and 20 pin tinyAVR parts! I've already got boards on standby for the 14-pin parts, in fact...

### EA-series
The EA-series is known only from it's product brief, and now some rather sparse headers. It looks like it is the first example of a new generation. Except for the whole lower maximum clock speed thing, that I don't get... It looks like it's internal oscillator is tinyAVR-like... According to the product brief it will be available only in 28, 32, and 48 pin packages, with up to 64k of flash - or as little as 8k (implying it is at least in part aimed at dragging the users of the old ATmega8 and such into the modern era, kicking and screaming), no Type D timer, but in exchange, there will be dual type A timers in all pincounts (confirmed by headers, with new 5-6-7 mappings for multiple ports) and it will feature the 12-bit true differential ADC with programmable gain amplifier that the tinyAVR 2-series has. The DAC appears to have some new functionality, albeit likely nothing huge. Event system looks to be getting significant changes with the laudable goal of making the channels all the same, which would make things like input capture easier but for the fact that people have written code for the current parts.

The most puzzling thing about these, though is that some of their specs like the oscillator seem to represent a step backwards. On the third hand, they will represent YET ANOTHER version of the NVM controller, being the first modern AVR to advertise NRWW and RWW sections of flash - but again, many would argue it's a step backwards- it is much more like the tinyAVR and megaAVR 0-series than the AVR Dx.  We can expect that programming over UPDI using SerialUPDI will be able to reach high baud rates - but the effective data rate will likely remain lower than the DX-series because of the smaller page size and additional commands required to write pagewise - RWW helps, but not by nearly as much as eliminating several latency periods per page and quadrupling the size of the page like the Dx did.

### DU-series (details unconfirmed)
The DU-series was described by a product brief which was almost immediately pulled down. It described something with DD-like pincounts, MVIO replaced with USB, and no TCD. One imagines that under the hood, TCD is repurposed for the 48 MHz USB reference clock, and the MVIO is levelshifting the USB data lines to 3.3v. It feature crystalless USB, the same flash options as the DD (except no 64k 14/20 pin). It also suggested a builtin USB programming interface of some sort, which is of course very exciting news - though all USB stuff will depend on availability of reference code to write the USB modules. We look forward to the release of more information, and, evnetually, the product. I predict the DU14 and DU20 will be extremely popular.

## Timeline
128k DA-series parts were released in April 2020, followed by 32k in early summer (though the initial release of the 32k parts suffered from a fatal flaw and was recalled, working ones were not available until the end of 2020), while the 64k parts became available late in summer of 2020, around the same time as the first AVR128DB-series parts became available, while the 32 snd 64k versions of the DB arriving in the first half of 2021. The AVR DD-series product brief was released in spring of 2020, but didn't become available until two years later in June of 2022, starting with 64k parts with 28 and 32 pins, and 16 and 32k parts with 14 and 20 pins. The other versions arrived in late 2022 - though as of writing the QFN package versions of the second tranch are still not available.

## Lots of errata
The silicon errata list in the initial versions of these parts - the DA-series in particular - is... longer than you may be used to if you were accustomed to the classic AVRs. The initial DB silicon looked bad too, but right after the release, a new silicon rev was added to the errata listing which fixed the worst of the bugs present in the "first" AVR128DB rev. All the parts I've gotten had the fixes, even the ones that arrived before they'd released the datasheet... Sadly, the AVR128DA never got a corresponding pack of fixes. In both cases, the smaller flash versions, released later, incorporated fixes that the 128k parts haven't gotten. If you've been working with the tinyAVR 0/1-series, many of these errata may be old friends of yours by now. A large number of those issues appear to have sailed through the new chip design process without a remedy - and five years after the release of those parts, new silicon errata were still being acknowledged in the tinyAVR 0/1 errata and DA/DB errata simultaneously - apparently issues that were discovered on the Dx-series and then found to impact the tinyAVR 0/1-series as well.

The DD series parts have done a *much* better job on the errata. There are still a few issues, but nothing like it was on the first round.

See [**our errata summary**](https://github.com/SpenceKonde/DxCore/blob/master/megaavr/extras/Errata.md) for more information.

## DD pricing
I'm sure you were all wondering about what the pricing of the DD-series parts would be like, and how it would compare to the existing Dx and tinyAVR parts.
Well, the answer is that it's closer to tinyAVR than DA/DB territory. Whether that means the extra peripherals on the DA/DB use a lot of silicon, or just have higher margins is a question only Microchip knows the answer to. Well, unless someone were to dissolve the encapsulation with nitric acid and measure the die size. Which I'm sure someone has done, but it's a near certainty that they haven't shared their findings, as the folks likely to do that are competitors looking to gain an advantage by understanding the architecture via die photography and the like.


| Pincount     | DA128 | DB128 | 64DA | 64DB |   64DD   | 32DA | 32DB |   32DD   |   16DD   | t321x | t322x | t160x | t161x | t162x |
|--------------|-------|-------|------|------|----------|------|------|----------|----------|-------|-------|-------|-------|-------|
| 14-pin  SOIC |   -   |   -   |   -  |   -  | **1.23** |   -  |  -   | **0.99** | **0.95** |   -   |  0.96 |  0.77 |  0.86 |  0.88 |
| 20-pin  SOIC |   -   |   -   |   -  |   -  | **1.54** |   -  |  -   | **1.35** | **1.25** |  1.21 |  1.23 |  1.06 |  1.10 |  1.16 |
| 20-pin  VQFN |   -   |   -   |   -  |   -  |      tbd |   -  |  -   | **1.08** | **0.98** |   -   |  1.01 |  0.81 |  0.75 |  0.95 |
| 28-pin  PDIP | 3.15  |  3.31 | 2.77 | 2.93 | **2.21** | 2.46 | 2.63 |      tbd |      tbd |   -   |   -   |   -   |   -   |   -   |
| 28-pin  SOIC | 1.89  |  2.06 | 1.71 | 1.83 | **1.56** | 1.56 | 1.71 | **1.42** | **1.35** |   -   |   -   |   -   |   -   |   -   |
| 28-pin  SSOP | 1.84  |  1.97 | 1.66 | 1.79 | **1.42** | 1.52 | 1.66 | **1.29** | **1.20** |   -   |   -   |   -   |   -   |   -   |
| 28-pin  VQFN |   -   |   -   |  -   |  -   | **1.31** |  -   |  -   |      tbd |      tbd |   -   |   -   |   -   |   -   |   -   |
| 32-pin  TQFP | 2.06  |  2.19 | 1.82 | 1.96 | **1.44** | 1.66 | 1.79 | **1.32** | **1.25** |   -   |   -   |   -   |   -   |   -   |
| 32-pin  VQFN | 2.04  |  2.19 | 1.82 | 1.96 | **1.45** | 1.68 | 1.82 |      tbd |      tbd |   -   |   -   |   -   |   -   |   -   |

This gives prices (where comparable parts exist) of a 20-27% savings vs the DB (the DA is only about 5% less than the DB). It's worth noting that the DD-28 finally brings a 28-pin VQFN to the product line, something I'm sure the product designers would have liked to see in the DA/DB parts. Nobody in production wants to use a TSSOP package unless they absolutely have to - those things are large (though notas large as a wide SOIC), and the QFN28 is markedly cheaper than the TSSOP too. TQFP vs QFN seems to be a wash, cost wise, across the whole DX series, and these are no exception. And since every time a part has been available with wide SOIC and QFN packages, the QFN has been cheaper, it's no surprise that there's a discount on the QFN version of the DD32 relative to the SOIC version, but the scale is surprising. There's a 27 cent premium on both the 32DD and 16DD, which is a premium of 25% and 27% respectivelty. On the AVR128DB28, that premium is just 9 cents, under 5%, though the premium on SOIC for the 20-pin tinyAVRs vs the QFN is also quite large.

The DD20's are, in general, oddly expensive. To make sense, the 64DD20 in QFN needs to land somewhere below $1.30 for people to not ignore it in favor of the oddly cheap 28-pin one in QFN. So I'm going to guess $1.20-1.25.

Prices are microchip direct qty 1 price for the item, industrial temp range (quantity discounts are significant - If this document was targeting industry folks instead of hobbyists, makers and maker-entrepreneurs, I'd be comparing the 5000+ price instead). The trend would be the same.
New pincounts and packages seem to be dribbling out, probably as production capacity allows. A VQFN version of of the smaller 32 and 28 pin DDs is inevitable, and they recently added a 16k DD20 in QFN. I'm more nervous about the AVR64DD20, after all, the ATtiny3216 didn't get the QFN package.

Notice that the price difference

I am working on an updated version of the core that will add the DD parts as fully supported. That support is in, but I have somehow introduced two bugs, one of which makes debugging impossible: Serial doesn't work. Again. And I don't have a hardware debugger nor would I know how to use one if i did,

## EA news

### We got initial Ex-series headers
[It has some surprises in it](EA_first_look.md)

### EA will be a big deal
In case there was ever any doubt about it... I just hope we'll see more than just the DA, DB, and DD parts with the nifty faster, better, voltage-independent core, and the word-write flash. Even if it does cost a good deal of flash endurance, it is incredibly convenient.

Core adaptation will mean pulling in the clock speed logic from megaTinyCore, and the fancy-ADC code, and it'll mean some tricky adaptations to Event (the way RTC and Pin events are generated is different, though the specific differencces also make the rest of Event simpler.