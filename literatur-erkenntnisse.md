# Literatur & Quellen – Erkenntnisse

Stand: 09.10.2026 · Masterarbeit 24-207 „Entwurf, Auslegung und Implementierung einer Systemarchitektur und eines Flugsteuerungssystems mit Basis-Autopiloten für das UAV Alexis“ (TH Wildau, Betreuer Prof. Köthe)

Kennzeichnung: **Vorgabe** = Aufgabenblatt/Kurs · **Befund** = aus Quelle gelesen · **Rechnung** = nachgerechnet (KI-gestützt, linear) · **Offen** = noch zu klären

---

## 1. Aufgabenblatt (Vorgabe)
- Umfang = AP 1–5 auf Alex' Aufgabenblatt: Vertrautmachen mit Alexis (Paul), FCU (Janik) und vorhandenen Modellen; neues UAV-Modell mit Struktur wie S15/Breezer; Flugregelung für **Kurs, Höhe, Geschwindigkeit** mit untergelagerten Funktionen. Weitere Vorgaben gibt es nicht.
- Kein Exposé-Abstimmungsprozess. Reihenfolge: erst Regler, dann Rest.

## 2. Alexis – Kenndaten (Befund, mit Widersprüchen)
| Größe | Wert | Quelle | Status |
|---|---|---|---|
| Konfiguration | Hochdecker, umgekehrtes V-Leitwerk (Ruddervators), Pusher | TU-Berlin-Unterlagen | gelesen |
| Spannweite / Fläche / Bezugstiefe | b = 4 m, S = 1,09 m², l_μ = 0,28 m | TU-Berlin-Unterlagen | gelesen |
| Masse | Modell 7,864 kg ↔ real ca. 11 kg | Unruh 2008; Elektronik 2011; Rabe 2013 | **Widerspruch**, wiegen |
| Schwerpunkt | 0,183 m hinter VK, Stabilitätsmaß ca. 3 % | Walde | prüfen |
| Standschub | 71 N ↔ 77 N | versch. Unterlagen | **Widerspruch** |
| Servos | 8 ↔ 10 | versch. Unterlagen | **Widerspruch** |
| Modellbasis | Aufgabenblatt: S15/Breezer ↔ Dossier: Talon ↔ lokal: Unruh 6-DoF | – | **Offen** (B07) |

## 3. Unruhs lineare Modelle (Auslegungspunkt 20 m/s, 100 m) – Befund
- Datei: `SA_5 Oliver Unruh/Lin_Modells/Auslegungspunkt/UAV_Linear_Modell_{laengs,seite}.mat`
- Zustände: längs **[V, α, q, Θ]**, seitlich **[β, p, r, Φ]**; Eingänge U1 HR, U2 QR, U3 SR, U4 Schub, U5 DLC
- Eigenwerte:
  - Anstellwinkelschwingung −11,53 ± 6,00j (ω₀ = 13,0 rad/s, D = 0,89)
  - Phygoide −0,065 ± 0,332j (ω₀ = 0,34 rad/s, D = 0,19)
  - Taumelschwingung −0,93 ± 4,03j (ω₀ = 4,1 rad/s, D = 0,22)
  - Rollpol −23,55 s⁻¹
  - **Spirale +0,047 s⁻¹ (instabil)**, Verdopplungszeit ca. 14,7 s
- Steuerwirksamkeiten: M_η = −153,1; L_ξ = −121,9; X_δF = 3,69; X_V = −0,106
- **Offen:** Trimmwerte im Linearmodell (UT: HR 0,0461, Schub 0,4027) weichen von `tp.mat` ab (HR 0,0453, SH 0,4908). Arbeitspunkt klären.
- Gültig nur für 7,864 kg und unvalidierte Aerodynamik.

## 4. FCU V3.0 (TN_FCU_001/002 v0.1, 17.09.2026) – Befund
- MCU STM32H723 (handschriftlich auf Aufgabenblatt: „F7“ → **Offen**, Janik)
- 8 PWM, 2 CAN, SBUS, Telemetrie 433 MHz
- Sensoren: LSM6DSR (IMU), LIS3MDL (Mag), BMP390 (Baro), BNO055, GPS mit IST8310, MS4525DO (Airspeed, **Bestückung offen**)
- EKF liest laut TN_FCU_002 nur IMU und Magnetometer. Baro/GPS/Pitot ohne dokumentierten Nutzer (**Offen**, Fundstelle zu EKF-Ausgängen fehlt)
- 8 PWM-Ausgänge ↔ 8–10 Servos + ESC → Kanalbelegung offen
- Toolchain: Simulink/Embedded Coder → STM32CubeIDE → FCU (Durchstich FCU → Ruder bereits erledigt)

## 5. DGLR-Kurs „Flugregelung für unbemannte und bemannte Luftfahrzeuge“ (Köthe 2023)
Ordner `Masterarbeit/DGLR_Kurs`, Foliensätze D1–D13 + Matlab/Simulink-Code.

| Satz | Inhalt | VFTE | Relevanz |
|---|---|---|---|
| D2 | Modell = „Fliegendes Labor“ (ZOHD Nano Talon), V-Leitwerk, Trimmung | – | Modellstruktur |
| D3 | Linearisierung mit `linmod`, Zustandstransformation A* = C·A·C⁻¹ | – | Linearisierung |
| D4 | Rolldämpfer, Rolllage-PI, Kurs-PI, Anti-Windup | L1, L2 | Seitenbewegung |
| D5 | Nickdämpfer, Längslage-PI, Höhenregler mit Pseudo-Differentiator | L6, L7 | Längsbewegung |
| D6 | Aktuator-/Sensormodelle (Servo T = 0,032 s, Motor 0,008 s), Tiefpass, Kalman/EKF | – | Modellerweiterung |
| D7 | Simulink → Code, Diskretisierung, Stateflow | Sim.-Import | Implementierung |
| D8 | Fahrt über Höhenruder (PIDT1) oder Schub (PI), Komplementärfilter | L5 | Geschwindigkeit |
| D9 | **Gesamtstruktur Autopilot**, TECS, Navigation, Abfangbogen | L8, L9, L12 | Zielarchitektur |
| D11–13 | Verkehrsflugzeuge (mit Luckner) | – | nur Kontext |

Kernaussagen (Befund):
- D9 F.4 = VFTE L12 = Zielstruktur: Kurs über Querruder, Geschwindigkeit über Schub, Höhe über Höhenruder.
- Höhenregler: $\Theta_{cmd} = K_H(\Delta H - \dot H)$; die Folie verweist selbst auf „nicht stimmige Einheiten“.
- Komplementärfilter: $\hat V = \frac{1}{s+K_V}a_x + \frac{K_V}{s+K_V}V_A$
- Schub → Fahrt (Talon, Phygoid-Näherung): $F_S = 18{,}59/(s+0{,}99)$. PI-Auslegung über Vorgabe $T = 1/(T_1 s+1)$.
- Anti-Windup beim Kursregler: **Zurücksetzen** statt Einfrieren (D4 F.18).
- Filterzeitkonstante Rolldämpfer 0,05 s (20 rad/s), aus echten Messwerten gewählt.

Anforderungen laut Folien, zitiert als SAE AS 94900 (**Norm nicht eingesehen**):
- Stabilität: D ≥ 0,3, Amplitudenreserve ≥ 6 dB, Phasenreserve ≥ 45°
- Rolllage: 80 % in 5 s; ruhige Luft < 1°; Böen 50 % < 10°
- Längslage: 90 % in 5 s; ruhige Luft < 0,5°; Böen 50 % < 5°
- Kurs: Überschwingen ≤ 1,5°; ruhige Luft < 0,5°; Böen 50 % < 5°
- Höhe: ±9,14 m / ±18,29 m

Auffälligkeiten in den Folien:
- D9 F.4: K_ΦI mit „−“ an der Summe (in D4 F.3 ohne)
- D9 F.4: p-Rückführung ohne K_ξp gezeichnet
- D9 F.7: „seitliche Ablage“/Δχ statt vertikal/Δγ
- D3 F.9: „B* = CA“ statt CB
- D5 F.7: zweimal „ohne Hängewinkel“
- D7 F.6: `dt=100; // 10 ms`

## 6. AlphaLink VFTE (alphalink-vfte.com) – Lessons
L1 Rollwinkel · L2 Kurs · L3/L4 RCAH Roll und Gierdämpfer · L5 V über Höhenruder (PID) · L6 Nicklage · L7 Höhe · L8 TECS · L9 TECS H/V · L10/L11 RCAH · **L12 Full Autopilot** (H → K_H(ΔH − Ḣ) → Θ_cmd mit PI und Nickdämpfer; V → PI → Schub; χ → PI → Φ_cmd → PI mit Rolldämpfer → ξ; Complementary Filter für V̂)

## 7. Recherchedossier S1–S9 (Auszug)
- **S1** Hopf, Dommaschk, Block, Reinfeld, Krachten, Worrmann, Cracau & Köthe (2020): *The flying lab for applied flight control and flight mechanics*, DLRK 2020, doi:10.25967/530237
  - Nano-Talon-UAXS, Simulink-Template, modifizierter PX4-Stack mit 100 Hz, HiL über CAN
  - = **Kursmodell des DGLR-Kurses**
  - Einschränkung: Mitautoren überschneiden sich mit den Verfassern der FCU-Berichte → keine unabhängige Quelle
- **S2/S3** Beard & McLain: successive loop closure. Bandbreitentrennung Faktor 5–10 = **Heuristik** (Matrix B15)
- **S5** PX4: TECS, Kopplung Höhe/Geschwindigkeit
- **S6** MathWorks: Trimmung/Linearisierung (`linearize` statt `linmod`)

## 8. Rechtsrahmen
- VO (EU) 2019/947, Unterkategorie A3 (geprüft). Keine Flugfreigabe aus der Arbeit ableiten.
