title: ALSA
date: 2019-07-07 00:10:10
author: Xavier
tags: - alsa
type: article

---

# amixer

```
root@TinaLinux:~# amixer controls
numid=105,iface=MIXER,name='Headphone Switch'
numid=106,iface=MIXER,name='Linein_detect Switch'
numid=74,iface=MIXER,name='Phoneout Mixer Left Output Mixer Switch'
numid=76,iface=MIXER,name='Phoneout Mixer MIC2Booststage Switch'
numid=75,iface=MIXER,name='Phoneout Mixer Right Output Mixer Switch'
numid=22,iface=MIXER,name='ADC input gain'
numid=18,iface=MIXER,name='ADC volume'
numid=25,iface=MIXER,name='ADCL Mux'
numid=24,iface=MIXER,name='ADCR Mux'
numid=51,iface=MIXER,name='AIF1 AD0L Mixer ADCL Switch'
numid=49,iface=MIXER,name='AIF1 AD0L Mixer AIF1 DA0L Switch'
numid=50,iface=MIXER,name='AIF1 AD0L Mixer AIF2 DACL Switch'
numid=52,iface=MIXER,name='AIF1 AD0L Mixer AIF2 DACR Switch'
numid=47,iface=MIXER,name='AIF1 AD0R Mixer ADCR Switch'
numid=45,iface=MIXER,name='AIF1 AD0R Mixer AIF1 DA0R Switch'
numid=48,iface=MIXER,name='AIF1 AD0R Mixer AIF2 DACL Switch'
numid=46,iface=MIXER,name='AIF1 AD0R Mixer AIF2 DACR Switch'
numid=44,iface=MIXER,name='AIF1 AD1L Mixer ADCL Switch'
numid=43,iface=MIXER,name='AIF1 AD1L Mixer AIF2 DACL Switch'
numid=42,iface=MIXER,name='AIF1 AD1R Mixer ADCR Switch'
numid=41,iface=MIXER,name='AIF1 AD1R Mixer AIF2 DACR Switch'
numid=13,iface=MIXER,name='AIF1 ADC timeslot 0 mixer gain'
numid=9,iface=MIXER,name='AIF1 ADC timeslot 0 volume'
numid=14,iface=MIXER,name='AIF1 ADC timeslot 1 mixer gain'
numid=10,iface=MIXER,name='AIF1 ADC timeslot 1 volume'
numid=11,iface=MIXER,name='AIF1 DAC timeslot 0 volume'
numid=12,iface=MIXER,name='AIF1 DAC timeslot 1 volume'
numid=104,iface=MIXER,name='AIF1IN0L Mux'
numid=103,iface=MIXER,name='AIF1IN0R Mux'
numid=102,iface=MIXER,name='AIF1IN1L Mux'
numid=101,iface=MIXER,name='AIF1IN1R Mux'
numid=56,iface=MIXER,name='AIF1OUT0L Mux'
numid=55,iface=MIXER,name='AIF1OUT0R Mux'
numid=54,iface=MIXER,name='AIF1OUT1L Mux'
numid=53,iface=MIXER,name='AIF1OUT1R Mux'
numid=17,iface=MIXER,name='AIF2 ADC mixer gain'
numid=15,iface=MIXER,name='AIF2 ADC volume'
numid=65,iface=MIXER,name='AIF2 ADL Mixer ADCL Switch'
numid=62,iface=MIXER,name='AIF2 ADL Mixer AIF1 DA0L Switch'
numid=63,iface=MIXER,name='AIF2 ADL Mixer AIF1 DA1L Switch'
numid=64,iface=MIXER,name='AIF2 ADL Mixer AIF2 DACR Switch'
numid=61,iface=MIXER,name='AIF2 ADR Mixer ADCR Switch'
numid=58,iface=MIXER,name='AIF2 ADR Mixer AIF1 DA0R Switch'
numid=59,iface=MIXER,name='AIF2 ADR Mixer AIF1 DA1R Switch'
numid=60,iface=MIXER,name='AIF2 ADR Mixer AIF2 DACL Switch'
numid=16,iface=MIXER,name='AIF2 DAC volume'
numid=69,iface=MIXER,name='AIF2INL Mux'
numid=71,iface=MIXER,name='AIF2INL Mux VIR switch aif2inl aif3'
numid=73,iface=MIXER,name='AIF2INL Mux switch aif2inl aif2'
numid=68,iface=MIXER,name='AIF2INR Mux'
numid=70,iface=MIXER,name='AIF2INR Mux VIR switch aif2inr aif3'
numid=72,iface=MIXER,name='AIF2INR Mux switch aif2inr aif2'
numid=67,iface=MIXER,name='AIF2OUTL Mux'
numid=66,iface=MIXER,name='AIF2OUTR Mux'
numid=57,iface=MIXER,name='AIF3OUT Mux'
numid=20,iface=MIXER,name='DAC mixer gain'
numid=19,iface=MIXER,name='DAC volume'
numid=97,iface=MIXER,name='DACL Mixer ADCL Switch'
numid=100,iface=MIXER,name='DACL Mixer AIF1DA0L Switch'
numid=99,iface=MIXER,name='DACL Mixer AIF1DA1L Switch'
numid=98,iface=MIXER,name='DACL Mixer AIF2DACL Switch'
numid=93,iface=MIXER,name='DACR Mixer ADCR Switch'
numid=96,iface=MIXER,name='DACR Mixer AIF1DA0R Switch'
numid=95,iface=MIXER,name='DACR Mixer AIF1DA1R Switch'
numid=94,iface=MIXER,name='DACR Mixer AIF2DACR Switch'
numid=77,iface=MIXER,name='HP_L Mux'
numid=78,iface=MIXER,name='HP_R Mux'
numid=38,iface=MIXER,name='LEFT ADC input Mixer LINEINL Switch'
numid=39,iface=MIXER,name='LEFT ADC input Mixer Lout_Mixer_Switch'
numid=34,iface=MIXER,name='LEFT ADC input Mixer MIC1 boost Switch'
numid=35,iface=MIXER,name='LEFT ADC input Mixer MIC2 boost Switch'
numid=37,iface=MIXER,name='LEFT ADC input Mixer PHONEN Switch'
numid=36,iface=MIXER,name='LEFT ADC input Mixer PHONEP-PHONEN Switch'
numid=40,iface=MIXER,name='LEFT ADC input Mixer Rout_Mixer_Switch'
numid=6,iface=MIXER,name='LINEINL/R to L_R output mixer gain'
numid=87,iface=MIXER,name='Left Output Mixer DACL Switch'
numid=86,iface=MIXER,name='Left Output Mixer DACR Switch'
numid=88,iface=MIXER,name='Left Output Mixer LINEINL Switch'
numid=92,iface=MIXER,name='Left Output Mixer MIC1Booststage Switch'
numid=91,iface=MIXER,name='Left Output Mixer MIC2Booststage Switch'
numid=89,iface=MIXER,name='Left Output Mixer PHONEN Switch'
numid=90,iface=MIXER,name='Left Output Mixer PHONEP-PHONEN Switch'
numid=4,iface=MIXER,name='MIC1 boost amplifier gain'
numid=26,iface=MIXER,name='MIC2 SRC'
numid=5,iface=MIXER,name='MIC2 boost amplifier gain'
numid=31,iface=MIXER,name='RIGHT ADC input Mixer LINEINR Switch'
numid=33,iface=MIXER,name='RIGHT ADC input Mixer Lout_Mixer_Switch'
numid=27,iface=MIXER,name='RIGHT ADC input Mixer MIC1 boost Switch'
numid=28,iface=MIXER,name='RIGHT ADC input Mixer MIC2 boost Switch'
numid=30,iface=MIXER,name='RIGHT ADC input Mixer PHONEP Switch'
numid=29,iface=MIXER,name='RIGHT ADC input Mixer PHONEP-PHONEN Switch'
numid=32,iface=MIXER,name='RIGHT ADC input Mixer Rout_Mixer_Switch'
numid=79,iface=MIXER,name='Right Output Mixer DACL Switch'
numid=80,iface=MIXER,name='Right Output Mixer DACR Switch'
numid=83,iface=MIXER,name='Right Output Mixer LINEINR Switch'
numid=85,iface=MIXER,name='Right Output Mixer MIC1Booststage Switch'
numid=84,iface=MIXER,name='Right Output Mixer MIC2Booststage Switch'
numid=82,iface=MIXER,name='Right Output Mixer PHONEP Switch'
numid=81,iface=MIXER,name='Right Output Mixer PHONEP-PHONEN Switch'
numid=23,iface=MIXER,name='SRC FUCTION'
numid=21,iface=MIXER,name='digital volume'
numid=1,iface=MIXER,name='headphone volume'
numid=2,iface=MIXER,name='lineout volume'
numid=7,iface=MIXER,name='phonein pre-amplifier gain'
numid=8,iface=MIXER,name='phonein(p-n) to L_R output mixer gain'
numid=3,iface=MIXER,name='phoneout volume'
```

```
amixer cset numid=1,iface=MIXER,name='headphone volume' 30
amixer set 'headphone volume' 80%
```

```
amixer cget numid=1,iface=MIXER,name='headphone volume'
amixer get 'headphone volume'
```

# arecord

arecord -D hw:M8,0 -r 16000 -f S16_LE -t raw -c 8 -d 10 speech8.pcm

# aplay

aplay -D hw:sndcodec,0 -f S16_LE -r 16000 -c 2 a.wav

beforming_ULA_asr_ch4_2_mic4_35mm_dirWavOff_vpOff.bin
AEC_ch8-2-ch4_2ref_v2_outgain4_d1_amth0.0002_2021_1_25.bin
