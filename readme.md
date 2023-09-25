# Guitar effects

How close can we get our guitar tone to a given recording?

* Possible to avoid having to model specific guitars and amps? (Might not know what was used anyway; prohibitively expensive and time-consuming.)
    * Different approach: use impulse response (many available on web for free), convolve with input to produce output
* NeuralPi example below works in real-time but that requires _a lot_ of work!
* Start with offline, any language and hardware.
* Use dynamic time warping to align guitar input to target recording?
    * First do coarse alignment: which section of recording corresponds to input?
* Use source separation to isolate the guitar track in the recording.
* There will be limits imposed by the guitar and amp e.g. no amount of audio processing could make an SSS strat sound like a guitar with humbuckers.
* Port to Rust?! (NeuralPi requires SIMD: only experimental support in Rust.)
* Could real-time performance be improved further by compressing the model?
    * Rather than set small weights to zero after training, [book](https://www.oreilly.com/library/view/hands-on-machine-learning/9781492032632/) suggests using L1-regularization during training to drive as many weights as possible to zero.


## TODO

* MVP:
    * Take any input (existing recording of guitar with clean tone).
    * Apply any effect (distortion) using any stack (Python?).
    * Save to file and play it back.
* Choose example song and isolate guitar part.
    * Record yourself trying to play that same part (short subsection).
* Read links above and see what can be adapted:
    * How do they define loss function?
    * What are their inputs and outputs?
* Gather more links, summarise here.
* Diagrams
    * Data preparation
    * Loss function
    * Model inference
* Audio interface:
    * Choose one
    * Get it working (see guitar input in browser/DAW)
* Tuner (see JS Rocks ... can it be improved upon e.g. smoothing, considering harmonics?)
* Parametric EQ
* Pipeline to isolate guitar samples of interest (or see what is available online?)

## Useful links

* [RTNeural](https://github.com/jatinchowdhury18/RTNeural): A lightweight neural network inferencing engine written in C++. This library was designed with the intention of being used in real-time systems, specifically real-time audio processing.
* [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
* [Librosa](https://librosa.org/) Audio and music processing in Python.
    * [Dynamic Time Warping](https://librosa.org/librosa_gallery/auto_examples/plot_music_sync.html) Align two audio signals in time domain (does it actually modify one or only identify corresponding samples?). Example plot doesn't look very good!

### JS Rocks

Guitar effects and cabinets using Web Audio API.

Uses impulse responses to model well-known amps and the reverb of several different spaces.

* [GitHub repo](https://github.com/vitaliy-bobrov/js-rocks)
* [Web app](https://js-rocks.web.app/stage)

### NeuralPi

NeuralPi is a guitar pedal using neural networks to emulate real amps and pedals on a Raspberry Pi 4.

* [GitHub repo](https://github.com/GuitarML/NeuralPi)
* [Blog post](https://towardsdatascience.com/neural-networks-for-real-time-audio-raspberry-pi-guitar-pedal-bded4b6b7f31)
* [YouTube demo](https://www.youtube.com/watch?v=_3zFD6h6Wrc)

### PedalNet

* [Original repo](https://github.com/teddykoker/pedalnet)
* [GuitarML fork](https://github.com/GuitarML/PedalNetRT)

### Model compression

* [Compressing and regularizing deep neural networks](https://www.oreilly.com/content/compressing-and-regularizing-deep-neural-networks/)

### Pedal PI

Raspberry Pi Zero based guitar pedal with real-time effects written in C.

* [Pedal PI article](https://www.electrosmash.com/pedal-pi)
* [Pedal PI kit](https://www.electrosmash.com/shop/product/pedal-pi-kit/)
* [Basics of Audio DSP in C](https://www.electrosmash.com/forum/pedal-pi/207-basics-of-audio-dsp-in-c-for-rapsberry-pi-zero?lang=en)

```C
// CC-by-www.Electrosmash.com Pedl-Pi open-source project
// clean.c effect pedal, the signal is read by the ADC and written again using 2 PWM signals. 
  
#include <stdio.h><br>#include <bcm2835.h> 
 
// Define Input Pins
#define PUSH1             RPI_GPIO_P1_08      //GPIO14
#define PUSH2             RPI_V2_GPIO_P1_38   //GPIO20 
#define TOGGLE_SWITCH     RPI_V2_GPIO_P1_32   //GPIO12
#define FOOT_SWITCH       RPI_GPIO_P1_10      //GPIO15
#define LED               RPI_V2_GPIO_P1_36   //GPIO16
 
uint32_t read_timer=0;
uint32_t input_signal=0;
  
uint8_t FOOT_SWITCH_val;
uint8_t TOGGLE_SWITCH_val;
uint8_t PUSH1_val;
uint8_t PUSH2_val;
  
 
//main program 
int main(int argc, char **argv)
{
    // Start the BCM2835 Library to access GPIO.
    if (!bcm2835_init())
    {printf("bcm2835_init failed. Are you running as root??\n"); return 1;}
 
 
    // Start the SPI BUS.
    if (!bcm2835_spi_begin())
    {printf("bcm2835_spi_begin failed. Are you running as root??\n"); return 1;}
  
    //define PWM mode   
    bcm2835_gpio_fsel(18,BCM2835_GPIO_FSEL_ALT5 ); //PWM0 signal on GPIO18    
    bcm2835_gpio_fsel(13,BCM2835_GPIO_FSEL_ALT0 ); //PWM1 signal on GPIO13    
    bcm2835_pwm_set_clock(2);                      // Max clk frequency (19.2MHz/2 = 9.6MHz)
    bcm2835_pwm_set_mode(0,1 , 1);                 //channel 0, markspace mode, PWM enabled. 
    bcm2835_pwm_set_range(0,64);                   //channel 0, 64 is max range (6bits): 9.6MHz/64=150KHz PWM freq.
    bcm2835_pwm_set_mode(1, 1, 1);                 //channel 1, markspace mode, PWM enabled.
    bcm2835_pwm_set_range(1,64);                   //channel 0, 64 is max range (6bits): 9.6MHz/64=150KHz PWM freq.
  
    //define SPI bus configuration
    bcm2835_spi_setBitOrder(BCM2835_SPI_BIT_ORDER_MSBFIRST);      // The default
    bcm2835_spi_setDataMode(BCM2835_SPI_MODE0);                   // The default
    bcm2835_spi_setClockDivider(BCM2835_SPI_CLOCK_DIVIDER_64);    // 4MHz clock with _64 
    bcm2835_spi_chipSelect(BCM2835_SPI_CS0);                      // The default
    bcm2835_spi_setChipSelectPolarity(BCM2835_SPI_CS0, LOW);      // the default
  
    uint8_t mosi[10] = { 0x01, 0x00, 0x00 }; //12 bit ADC read channel 0. 
    uint8_t miso[10] = { 0 };
  
    //Define GPIO pins configuration
    bcm2835_gpio_fsel(PUSH1, BCM2835_GPIO_FSEL_INPT);           //PUSH1 button as input
    bcm2835_gpio_fsel(PUSH2, BCM2835_GPIO_FSEL_INPT);           //PUSH2 button as input
    bcm2835_gpio_fsel(TOGGLE_SWITCH, BCM2835_GPIO_FSEL_INPT);   //TOGGLE_SWITCH as input
    bcm2835_gpio_fsel(FOOT_SWITCH, BCM2835_GPIO_FSEL_INPT);     //FOOT_SWITCH as input
    bcm2835_gpio_fsel(LED, BCM2835_GPIO_FSEL_OUTP);             //LED as output
  
    bcm2835_gpio_set_pud(PUSH1, BCM2835_GPIO_PUD_UP);           //PUSH1 pull-up enabled   
    bcm2835_gpio_set_pud(PUSH2, BCM2835_GPIO_PUD_UP);           //PUSH2 pull-up enabled 
    bcm2835_gpio_set_pud(TOGGLE_SWITCH, BCM2835_GPIO_PUD_UP);   //TOGGLE_SWITCH pull-up enabled 
    bcm2835_gpio_set_pud(FOOT_SWITCH, BCM2835_GPIO_PUD_UP);     //FOOT_SWITCH pull-up enabled 
  
    while(1) //Main Loop
    {
        //Read the PUSH buttons every 50000 times (0.25s) to save resources.
        read_timer++;
        if (read_timer==50000)
        {
            read_timer=0;
            uint8_t PUSH1_val = bcm2835_gpio_lev(PUSH1);
            uint8_t PUSH2_val = bcm2835_gpio_lev(PUSH2);
            TOGGLE_SWITCH_val = bcm2835_gpio_lev(TOGGLE_SWITCH);
            uint8_t FOOT_SWITCH_val = bcm2835_gpio_lev(FOOT_SWITCH);
            bcm2835_gpio_write(LED,!FOOT_SWITCH_val); //light the effect when the footswitch is activated.
        }
    
        //read 12 bits ADC
        bcm2835_spi_transfernb(mosi, miso, 3);
        input_signal = miso[2] + ((miso[1] & 0x0F) << 8); 
    
        //**** CLEAN EFFECT ***///
        //Nothing to do, the input_signal goes directly to the PWM output.
    
        //generate output PWM signal 6 bits
        bcm2835_pwm_set_data(1,input_signal & 0x3F);
        bcm2835_pwm_set_data(0,input_signal >> 6);
    }
  
    //close all and exit
    bcm2835_spi_end();
    bcm2835_close();
    return 0;
}
```
