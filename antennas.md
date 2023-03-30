# links

https://www.antenna-theory.com/

# frequency bands

Frequency Band Name            | Frequency Range                 | Wavelength (Meters)     | Application
Extremely Low Frequency (ELF)  | 3-30 Hz                         | 10,000-100,000 km       | Underwater Communication
Super Low Frequency (SLF)      | 30-300 Hz                       | 1,000-10,000 km         | AC Power (though not a transmitted wave)
Ultra Low Frequency (ULF)      | 300-3000 Hz                     | 100-1,000 km
Very Low Frequency (VLF)       | 3-30 kHz                        | 10-100 km               | Navigational Beacons
Low Frequency (LF)             | 30-300 kHz                      | 1-10 km                 | AM Radio
Medium Frequency (MF)          | 300-3000 kHz                    | 100-1,000 m             | Aviation and AM Radio
High Frequency (HF)            | 3-30 MHz                        | 10-100 m                | Shortwave Radio
Very High Frequency (VHF)      | 30-300 MHz                      | 1-10 m                  | FM Radio
Ultra High Frequency (UHF)     | 300-3000 MHz                    | 10-100 cm               | Television, Mobile Phones, GPS
Super High Frequency (SHF)     | 3-30 GHz                        | 1-10 cm                 | Satellite Links, 5-7GHz WIFI, UWB
Extremely High Frequency (EHF) | 30-300 GHz                      | 1-10 mm                 | Astronomy, Vehicle Radar, mmWave 5G
Visible Spectrum               | 400-790 THz (4*10^14-7.9*10^14) | 380-750 nm (nanometers) | Human Eye


# Radiation pattern

* isotropic - Like a sphere - perfectly radiation in all spatial angles the same. Only theoretical.
* omnidirectional - same in all directions (in the azimuth plane)
* Typically shows in elevation graph (θ) and azimuth graph (ф)
    * elevation/polar angle (θ)
        * 0-degrees is straight up
        * 90-degress is at orizontal plane
        * 180-degress is straight down.
    * azimuth (ф)
        * 0-degress is north
        * 90-degress is east, 180-south, 270 - west

## spherical co-ordinates

* R
    * the magnitude of the distance between the origin and the point
    * always positive
* (θ) polar angle
    * angle between the z-axis and the vector from the origin to the point
    * ranges from 0 to 180 degrees
* (ф) azimuth angle
    * angle between the x-axis and the projection of the point onto the x-y plane
    * ranges from 0 to 360 degrees

# Field Regions

Terms:

* E-field - electric field
* H-field - magmetic field
* R - distance from antenna
* D - dimension (length/breadth) of antenna
* λ - wavelength


* Far (Franhauffer)
    * Really where the action happens
    * Distance after which radiation pattern doesn't change much
    * Like plane waves, E & H are orthogonal to each other and to direction of wave.
        * E & H are in phase.
    * Is defined by
        ```
            R > 2(D**2)      Ensures power is parallel at given R
                -------
                  λ

            R >> D
            R >> λ
        ```
    * Dominated by radiated fields
    * In general
        * E & H field die at a rate of 1/R (inversely proportional to distance)
        * power density dies at rate of 1/(R**2) (inversely proportional to square of distance)

* Reactive Near field
    * In immediate vicinity of Antenna
    * E & H are out of phase by 90-degress (Reactive fields)

* Radiating near field / Fresnel region
    * Between near and far fields
    * the reactive fields are not dominate; the radiating fields begin to emerge
    * Shape of radiation pattern may vary with distance

# directivity

* It is a measure of how 'directional' an antenna's radiation pattern is.
* An antenna that radiates equally in all directions would have effectively
  zero directionality, and the directivity of this type of antenna would be
  1 (or 0 dB).
* D in simple english goes like this:
    ```
          D =                       1                (Max power)
                -------------------------------------
                Avg power radiated over all direction
    ```
* If avg get lesser and lesser, (one sample is v.high, but rest of spatial samples are low),
  then the antenna is very directional

## number to decibel conversion

Antenna Type               | Typical Directivity | Typical Directivity (dB)
Short Dipole Antenna       | 1.5                 | 1.76
Half-Wave Dipole Antenna   | 1.64                | 2.15
Patch (Microstrip) Antenna | 3.2-6.3             | 5-8
Horn Antenna               | 10-100              | 10-20
Dish Antenna               | 10-10,000           | 10-40

# Efficiency

* This gives for tx antenna.
```
    Efficient =  Power radiated
                 --------------
                 Power supplied
```


# Antenna Gain

* The term Antenna Gain describes how much power is transmitted in the direction of peak
  radiation to that of an isotropic source
* Combines directivity and electrical efficiency
    * As a transmitting antenna, the gain describes how well the antenna converts input power
      into radio waves headed in a specified direction.
    * As a receiving antenna, the gain describes how well the antenna converts radio waves
      arriving from a specified direction into electrical power.
    * When no direction is specified, gain refers to the peak value of the gain
* Directivity is a bit theorteical, while Gain is what is actually measured - so its typically what is
  often quoted.
* Mathematically
        ```
            G = efficienty x Directivity
        ```
* Directivity is always from 0Db to ∞ (1 to ∞)
* Gain can be -ve in db (fractional to ∞). This is because efficiency can be bad.

# Beamwidths and side lobes

* Characterizes radiation pattern in addition to directivity
* The Half Power Beamwidth (HPBW)
    * the angular separation in which the magnitude of the radiation pattern
      decrease by 50% (or -3 dB) from the peak of the main beam.
* Null to Null Beamwidth.
    * the angular separation from which the magnitude of the radiation pattern
      decreases to zero (negative infinity dB) away from the main beam.
* The sidelobes
    * are smaller beams that are away from the main beam.
    * These sidelobes are usually radiation in undesired directions
      which can never be completely eliminated.
* the Sidelobe Level
    * another important parameter used to characterize radiation patterns.
    * the maximum value of the sidelobes (away from the main beam).

# EIRP

* Effective/Equivalent Isotropic Radiated Power
    * The measured radiated power in a single direction is known as the EIRP.
    * Typically, for an antenna radiation pattern measurement, if a single value
      of EIRP is given, this will be the maximum value of the EIRP over
      all measured angles.
* EIRP can also be thought of as the amount of power a perfectly isotropic
  antenna would need to radiate to achieve the measured value.
    * This is just techie way of saying, I picked the highest power point!
* The EIRP can be related to the power transmitted from the radio (P_t),
  the cable losses (possibly including antenna mismatch) L,
  and the antenna gain (G) by:
  ```
  EIRP = P_t - L + G
  ```
  * Typically L is very small and neglected.
