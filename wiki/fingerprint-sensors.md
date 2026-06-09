# Fingerprint Sensors

**Summary**: The 5 fingerprint sensor technologies and 3 acquisition modes — how fingerprint images are physically captured.

**Pre-Req**: [[biometric-system-modules]], [[image-processing-basics]]

**Sources**: Unit2_Biometrics(1).pptx

**Last updated**: 2026-05-10

#biometrics

---

## Sensor Technologies

### 1. FTIR — Frustrated Total Internal Reflection

**Physics**: Light undergoes total internal reflection inside a prism when the angle of incidence exceeds the critical angle. A fingerprint ridge touching the prism surface **frustrates** this reflection — light scatters out rather than reflecting — creating a dark pixel. Valleys don't touch → total internal reflection → bright pixels.

**Setup**: LED illuminates prism → CCD camera below captures scattered light pattern.

**Limitation**: Large focal length makes the optical path difficult to miniaturize — compact form factor is challenging.

(source: Unit2_Biometrics(1).pptx, Fig 2.12)

---

### 2. Capacitance (Solid-State)

**Physics**: An array of microscopic capacitor plates (electrodes) is embedded in a chip. The fingerprint skin acts as the second electrode. Ridges (contact) and valleys (air gap) produce different capacitances:
- Ridge over electrode → C_ridge (small air gap → higher C)
- Valley over electrode → C_valley (larger air gap → lower C)
- C_stray accounts for parasitic capacitance

The capacitance map is read out as an image.

**Advantages**:
- Small form factor → embeddable in laptops, smartphones, ATMs
- More common than FTIR in consumer devices

(source: Unit2_Biometrics(1).pptx)

---

### 3. Ultrasound Reflection

**Physics**: A sender generates high-frequency acoustic pulses. A receiver captures echoes bouncing off the fingerprint surface. The echo timing and amplitude maps to the ridge/valley topography.

**Advantages**: Resilient to surface contamination (dirt, oil, sweat) — signal penetrates through surface debris.

**Disadvantages**: Expensive; not suited for mass production. Primarily used in high-security applications.

---

### 4. Temperature Differential

**Physics**: Uses a **pyroelectric material** that generates an electrical current in response to temperature change (not absolute temperature). 

- Ridges contact the sensor surface → smaller temperature differential (heat transfers quickly)
- Valleys don't contact → larger temperature differential (air is insulating)

The current difference between ridge/valley contact produces the fingerprint image.

---

### 5. Piezoelectric Effect

**Physics**: Piezoelectric material generates an electrical charge when subjected to mechanical stress. The fingerprint ridge pattern exerts varying pressure across the sensor surface.

- Measures pressure, acceleration, or strain → converts to electrical charge
- Highly sensitive, very small form factor
- Suited for embedding in everyday objects (keyboards, pens)

---

## Acquisition Modes

Three ways a user places their finger on a sensor (source: Unit2_Biometrics(1).pptx, Fig 2.13):

| Mode | How | Use case |
|---|---|---|
| **Plain (flat)** | Place finger flat on sensor surface | Most common; consistent contact |
| **Rolled** | Roll finger nail-to-nail across sensor | Captures full fingerprint including sides; used in forensics/law enforcement |
| **Sweep** | Swipe finger across a narrow sensor; slices (~3mm wide) are stitched together | Used in thin sensors (laptop touchpads, phone-side sensors) |

Sweep sensors reconstruct a full image from multiple narrow strips in real time.

---

## Sensor Evolution

| Era | Development |
|---|---|
| Early | Large optical (FTIR) units; only for dedicated terminals |
| Mid | Compact capacitive sensors → laptop integration |
| 2005 | TBS 3D sensor — captures fingerprint topography in 3D |
| 2010 | Safran touchless sensor — no physical contact required |
| Present | Under-display optical/ultrasound sensors in smartphones |

---

## Related Pages

- [[fingerprint-feature-extraction]] — what happens to the image after capture
- [[fingerprint-recognition]] — full fingerprint recognition pipeline
- [[biometric-system-modules]] — sensor sits in module 1
- [[image-processing-basics]] — image types produced by these sensors
