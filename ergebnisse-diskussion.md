# Ergebnisse & Diskussion

Stand: 09.10.2026

## 1. Eigene Ergebnisse (Alexis)
- **Noch keine.** Reglerentwurf an Alexis beginnt am Wochenende 10./11.10.2026.

## 2. Vorab-Analysen (Rechnung, KI-gestützt, linear – nicht als Ergebnis zitieren)
Nachrechnung der DGLR-Kursauslegung (Talon, 17 m/s, reduzierte Modelle, ohne Aktuatoren und Totzeit):

| Kreis | Kurswert | Durchtritt | PM / GM | Verhalten |
|---|---|---|---|---|
| Rolldämpfer (T_p = 0,05 s) | k_ξp = −0,0025 | – | – | Rollpol −12,4 → −16,2 ± 1,2j |
| Rolllage-PI | −0,06693 − 0,09972/s | 2,0 rad/s | 45° / ∞ | ca. 35 % Überschwingen |
| Kurs (nur P) | k_χΦ = 1 | 0,69 rad/s | 85° / 15,5 dB | kein Überschwingen |
| Nickdämpfer (T_q = 0,02 s) | k_ηq = −0,132 | 21 rad/s | 106° / ∞ | AS → −18,1 ± 20,4j |
| Längslage-PI (Skript) | −1 − 7,618/s | 5,0 rad/s | 46° / 43 dB | 34 % Überschwingen |
| Längslage-PI (Code) | ≈ −0,65 − 0,29/s | 2,0 rad/s | 86° / ∞ | 12 % Überschwingen |

- Kaskadenabstand im Kurs: Rolle/Kurs ≈ 2,9; Dämpfer/Nicklage ≈ 4,2 → die Heuristik 5–10 (S3) wird unterschritten. Für Alexis Abstand nachweisen.
- Eigene Herleitung (prüfen): $\tau_H = 1\,\mathrm{s} + 1/(V K_H)$ → Alexis, 20 m/s, K_H = 0,05 → τ_H ≈ 2 s
- Erste Schätzung Schub → Fahrt Alexis: K_S ≈ 3,69, a ≈ 0,106 → bei T₁ = 2 s: K_P ≈ 0,136, K_I ≈ 0,0144 (Näherung schwach wegen Phygoide D = 0,19)

## 3. Diskussionsmaterial: Befunde im Kurs-Code `controllermodel.slx` (Lehrbeispiel, mit Alex klären)
- **F1:** Fahrtregler-Nenner auf [1 −1,819 0,8187] gerundet → Pol z = 1,0016 (instabil, s ≈ +0,16 s⁻¹), I-Anteil ausgelöscht. Im linearen Talon-Kreis Pol bei s ≈ +0,038 s⁻¹ (Verdopplung ca. 18 s)
- **F2:** Trimmwert als InitialStates (Direktform II) erscheint nicht am Ausgang
- **F3:** Reset „Either“ am Signal Regler (1 → 2) löst nie aus → Θ-Integrator windet in Z1/Z2 auf
- **F4:** Auto-Modus ohne Rolllageregelung, kein Anti-Windup, Lidar ungenutzt
- Bedeutung für die Arbeit: Implementierungsregeln (Methodik §5) begründen; Moduswechsel und Langzeitlauf als Testfälle

## 4. Diskussionspunkte für später
- Instabile Spirale bei Alexis → Rolllagekreis in allen Automatikmodi geschlossen halten
- Modellunsicherheit (Masse, Schwerpunkt, Aerodynamik) → Robustheitsnachweis über mehrere Massenfälle
- Fahrtschätzung hängt am Pitot (Bestückung offen); GPS-Geschwindigkeit ist kein Ersatz
- Quellenunabhängigkeit: S1 und die FCU-Berichte haben teils dieselben Autoren

## 5. Offene Fragen
- **Alex:**
  - controllermodel als Vorlage? F1–F3 bekannt?
  - AS-94900-Werte als Abnahmekriterien?
  - Was heißt „Struktur wie S15/Breezer“?
  - a_x schwerekompensiert?
  - Regler- und Sensorraten auf der FCU?
  - Genaues Abgabedatum, Tiefe der Implementierung?
- **Janik:** EKF-Ausgänge, MCU (H723/F7), Kanalbelegung
- **Paul:** Masse wiegen, Schwerpunkt, Pitot verbaut?
