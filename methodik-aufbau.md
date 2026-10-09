# Methodik & Aufbau

Stand: 09.10.2026 · Abgabe angenommen 31.12.2026 (genaues Datum noch bestätigen)

## 1. Vorgehen (Vorschlag)
1. Modell: Unruhs lineare Modelle als Vorentwurfsbasis; neues Modell (Struktur S15/Breezer bzw. Talon-Template) parallel aufbauen
2. Trimmung: `lsqnonlin` mit freien Variablen [α, η, δF], Residuen [R_x, R_z, M_y] = 0
3. Linearisierung: `linmod`/`linearize` um den Trimmpunkt, Transformation A* = C·A·C⁻¹, B* = C·B (C quadratisch, invertierbar)
4. Kaskadenauslegung von innen nach außen:
   - Dämpfer (Wurzelortskurve, negatives Vorzeichen) → Lage-PI (pidTuner/Optimierung) → Kurs/Höhe/Fahrt
5. Prüfung: Margins, Sprungantworten, volles lineares Modell (inkl. Spirale), danach nichtlineare Simulation/VFTE
6. Implementierung: Simulink → Embedded Coder → STM32CubeIDE → FCU
7. Validierung nach Testmatrix (Dossier §9.2, T01–T13)

## 2. Zielarchitektur (nach DGLR D9 F.4 / VFTE L12)
- Höhe: $\Theta_{cmd} = \mathrm{sat}\big(K_H(\Delta H - \dot{\hat H})\big)$, $\hat H$ über Tiefpass, $\dot{\hat H}$ über Pseudo-Differentiator $s/(T_H s+1)$
- Längslage: $\eta_{cmd} = \mathrm{sat}\big(K_{\Theta P}e_\Theta + K_{\Theta I}\int e_\Theta - K_{\eta q}q_f\big)$
- Fahrt: $\delta_{F,cmd} = \mathrm{sat}\big(K_{VP}e_V + K_{VI}\int e_V\big)$ mit Komplementärfilter
- Kurs: $\Phi_{cmd} = \mathrm{sat}\big(K_{\chi P}e_\chi + K_{\chi I}\int e_\chi\big)$
- Rolllage: $\xi_{cmd} = \mathrm{sat}\big(K_{\Phi P}e_\Phi + K_{\Phi I}\int e_\Phi - K_{\xi p}p_f\big)$
- Alle Integratoren mit Anti-Windup. TECS nur als begründete Alternative diskutieren.

## 3. Übertragungsfallen Kurs (Talon) → Alexis
| | Talon (Kurs) | Alexis (Unruh) |
|---|---|---|
| Zustände längs | [q, α, V, Θ] | [V, α, q, Θ] |
| Zustände seitlich | [r, β, p, Φ] | [β, p, r, Φ] |
| AS-Näherung | `A_LB(1:2,1:2)` | `A([3 2],[3 2])`, `B([3 2],1)` |
| Roll-Näherung | `A_SB(3:4,3:4)` | `A([2 4],[2 4])`, `B([2 4],2)` |
| M_η / L_ξ | −118,9 / −318,4 | −153,1 / −121,9 (gleiche Vorzeichen → negative Dämpfergains) |
| Spirale | −0,11 (stabil) | +0,047 (instabil) |

Weitere Punkte:
- Drehratenderivative im Kurs nur durch V geteilt (nicht b/2V bzw. c/2V) → Unruhs Konvention prüfen
- V-Leitwerksmischung Kurs: η_r = η − ζ, η_l = η + ζ. Alexis hat ein **umgekehrtes** V-Leitwerk, Vorzeichen prüfen (B12)
- a_x im Komplementärfilter evtl. nicht schwerekompensiert ($\dot V \approx a_x - g\sin\Theta$) → offen

## 4. Anforderungen (Vorschläge, mit Alex abzustimmen)
| Nr. | Anforderung |
|---|---|
| R01 | Modell parametrisiert |
| R02 | Kurs, Höhe, Geschwindigkeit geregelt (Zahlen: Kandidaten aus AS 94900, s. Literatur) |
| R03 | Untergelagerte Kreise stabilisieren alle Eigenformen inkl. Spirale |
| R04 | Aktuatorgrenzen berücksichtigt |
| R05 | Moduswechsel/Failsafe definiert |
| R06 | Systemarchitektur dokumentiert |
| R07 | Echtzeitfähigkeit |
| R08 | Robustheit (Masse 7,9 ↔ 11 kg, Schwerpunkt) |
| R09 | Reproduzierbarkeit |

## 5. Implementierungsregeln (Lehre aus Kurs-Code, Vorschlag)
- Gains als benannte Parameter (Data Dictionary/`init.m`), keine handgerundeten Koeffizienten
- Discrete-PID-Block mit Anti-Windup (Clamping), externem Reset und Tracking
- Koeffiziententest: `abs(roots(den)) <= 1`, Integratorpole exakt bei 1, auch in `single`
- Stoßfreie Umschaltung: Reset bei Moduswechsel, I-Anteil = aktueller Stellwert − P-Anteil
- Trimm als Konstante nach dem Regler addieren
- Zusätzliche Tests: Aufschalten im Trimm, Moduswechsel, Langzeitlauf ≥ 5 min

## 6. Zeitplan Wochenende 09.–11.10.2026
- **Fr:** t1 Ladetest Unruh/S15 · t2 lineare Modelle laden · t3 numerische Linearisierung (Reproduktionstest mit Kursordner `Linearisierung`) · t4 Übertragungsfunktionen Alexis
- **Sa (bis 16 Uhr):** we1 Nickdämpfer · we2 Nicklage-PI · we3 Übung η-Regler · we4 Höhenregler (Startwert τ_H ≈ 2 s)
- **So:** we5–we7 AlphaLink-Einführung, Lesson 1 · we8 Rolldämpfer/Rolllage an Alexis · we9 Logbuch · optional we10 Kurs, we11 Fahrt

## 7. Arbeitsregeln
- Aerosonde-Übung (Weg B) = nur Übung, kein Ergebnis der Arbeit
- KI-Beiträge kennzeichnen, Logbuch mit Datum führen
- Keine Parameter, Messwerte oder Quellen erfinden
