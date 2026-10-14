# Hauptprogramm PolyScope – Version 5 (lesbar)
# برنامهٔ اصلی PolyScope – نسخهٔ ۵ (خواناتر)

Gleiche Logik wie Version 4, aber mit **verständlichen deutschen Namen**. Dazu kommen Selbstzentrierung und Lochplatte (`nut_versatz`).
همان منطق نسخهٔ ۴، با **اسم‌های آلمانی قابل فهم**. خودوسط‌شدن و برد سوراخ‌دار (`nut_versatz`) هم اضافه شده‌اند.

Dazu gehört die Datei `funktionen_v5_lesbar.script`, die im ersten Script-Knoten geladen wird. Nicht am Roboter getestet; erster Lauf mit 10–20 %.
فایل `funktionen_v5_lesbar.script` هم لازم است و در اولین گرهٔ Script بار می‌شود. روی ربات امتحان نشده؛ اولین اجرا با ۱۰ تا ۲۰ درصد.

**Was ist anders als Version 4?**
- Statt einer Liste `typen` mit Spaltennummern (`typen[j][2]`) gibt es **eine Liste pro Eigenschaft** (`greif_tiefe[typ_nr]`). So sieht man sofort, was gemeint ist.
- Alle Variablen haben sprechende Namen, zum Beispiel `fehler_code` statt `erg` und `teil_x` statt `komp_x`.
- Die Sicherheitshöhe ist eine eigene Variable `sicher_hoehe`, statt überall die Zahl `0.06`.

**چه فرقی با نسخهٔ ۴ دارد؟**
- به‌جای یک لیست `typen` با شمارهٔ ستون (`typen[j][2]`)، **برای هر ویژگی یک لیست** هست (`greif_tiefe[typ_nr]`). این‌طوری فوراً معلوم است منظور چیست.
- همهٔ متغیرها اسم واضح دارند، مثلاً `fehler_code` به‌جای `erg` و `teil_x` به‌جای `komp_x`.
- ارتفاع امن یک متغیر جدا به اسم `sicher_hoehe` است، به‌جای عدد `0.06` در همه‌جا.

---

## 1. Programmbaum

```
Programm
│
├─ BeforeStart
│   ├─ [Script: Datei]  funktionen_v5_lesbar
│   ├─ [Script]  rq_reset()
│   ├─ [Script]  rq_activate_and_wait()
│   ├─ [Script]  rq_set_speed(128)
│   │
│   │   # --- A: Teiletypen. Eine Spalte pro Typ, kürzestes Teil zuerst.  <<< EINLERNEN ---
│   ├─ [Zuweisung]  breite        = [0.018, 0.036, 0.054]
│   ├─ [Zuweisung]  greifkraft    = [100, 120, 120]
│   ├─ [Zuweisung]  greif_tiefe   = [-0.012, -0.012, -0.015]
│   ├─ [Zuweisung]  einrast_tiefe = [-0.012, -0.012, -0.015]
│   ├─ [Zuweisung]  zieh_kraft    = [40.0, 60.0, 60.0]
│   ├─ [Zuweisung]  druck_kraft   = [40.0, 60.0, 60.0]
│   ├─ [Zuweisung]  zieh_weg      = [0.012, 0.012, 0.012]
│   ├─ [Zuweisung]  finger_pos    = [160, 140, 115]
│   ├─ [Zuweisung]  nut_versatz   = [0.0, 0.0, 0.0]
│   ├─ [Zuweisung]  anzahl_typen  = get_list_length(finger_pos)
│   │
│   │   # --- B: Greifer und Tasten ---
│   ├─ [Zuweisung]  kleinste_breite    = 0.018
│   ├─ [Zuweisung]  finger_toleranz    = 5
│   ├─ [Zuweisung]  extra_oeffnung     = 20
│   ├─ [Zuweisung]  halbe_fingerbreite = 0.008
│   ├─ [Zuweisung]  tast_hoehe         = -0.015
│   ├─ [Zuweisung]  sicher_hoehe       = 0.06
│   ├─ [Zuweisung]  greifer_drehung    = 0.0
│   │
│   │   # --- C: Platte  <<< EINLERNEN ---
│   ├─ [Zuweisung]  reihen_y            = [0.0, 0.080]
│   ├─ [Zuweisung]  reihen_abstand      = 0.080
│   ├─ [Zuweisung]  start_x_rechts      = 0.250
│   ├─ [Zuweisung]  suchweg_erstes_teil = 0.100
│   ├─ [Zuweisung]  max_reihen          = 4
│   │
│   │   # --- D: Schrank  <<< EINLERNEN ---
│   ├─ [Zuweisung]  schienen_y      = [0.0, 0.080, 0.160]
│   ├─ [Zuweisung]  schienen_ende_x = [0.300, 0.300, 0.300]
│   ├─ [Zuweisung]  schienen_belegt = [0.0, 0.0, 0.0]
│   ├─ [Zuweisung]  anzahl_schienen = get_list_length(schienen_y)
│   ├─ [Zuweisung]  abstand_teile   = 0.0
│   ├─ [Zuweisung]  versatz_x       = [0.0, 0.002, -0.002]
│   │
│   │   # --- E: Arbeitsvariablen (Startwerte, nicht ändern) ---
│   ├─ [Zuweisung]  reihe              = 0
│   ├─ [Zuweisung]  weitere_reihen     = True
│   ├─ [Zuweisung]  reihen_gefunden    = 0
│   ├─ [Zuweisung]  teile_in_reihe     = 0
│   ├─ [Zuweisung]  reihe_fertig       = False
│   ├─ [Zuweisung]  erstes_teil        = True
│   ├─ [Zuweisung]  typ_nr             = 0
│   ├─ [Zuweisung]  typ_test           = 0
│   ├─ [Zuweisung]  typ_gemessen       = 0
│   ├─ [Zuweisung]  suche_status       = 0
│   ├─ [Zuweisung]  kontakt_x          = 0.0
│   ├─ [Zuweisung]  kante_x            = 0.0
│   ├─ [Zuweisung]  kante_erwartet     = 0.0
│   ├─ [Zuweisung]  such_start_x       = 0.0
│   ├─ [Zuweisung]  such_weg           = 0.0
│   ├─ [Zuweisung]  test_x             = 0.0
│   ├─ [Zuweisung]  teil_x             = 0.0
│   ├─ [Zuweisung]  teil_y             = 0.0
│   ├─ [Zuweisung]  schiene_nr         = 0
│   ├─ [Zuweisung]  ablage_x           = 0.0
│   ├─ [Zuweisung]  ablage_y           = 0.0
│   ├─ [Zuweisung]  ziel               = p[0, 0, 0, 0, 0, 0]
│   ├─ [Zuweisung]  fehler_code        = 0
│   ├─ [Zuweisung]  versuche_entnehmen = 0
│   ├─ [Zuweisung]  versuch_einsetzen  = 0
│   ├─ [Zuweisung]  einsetz_status     = 0
│   └─ [Zuweisung]  sitzt_fest         = False
│
├─ Robot Program
│   ├─ [MoveJ]
│   │   └─ [Wegpunkt fest]  Home
│   │
│   │   # --- F: Reihe für Reihe ---
│   ├─ [Schleife solange]  weitere_reihen and reihe < max_reihen
│   │   ├─ [Script]  teil_y = reihen_position(reihen_y, reihen_abstand, reihe)
│   │   ├─ [Zuweisung]  schiene_nr = reihe
│   │   ├─ [Zuweisung]  kante_erwartet = start_x_rechts
│   │   ├─ [Zuweisung]  typ_nr = 0
│   │   ├─ [Zuweisung]  erstes_teil = True
│   │   ├─ [Zuweisung]  reihe_fertig = False
│   │   ├─ [Zuweisung]  teile_in_reihe = 0
│   │   │
│   │   │   # --- G: Teil für Teil, von rechts nach links ---
│   │   ├─ [Schleife solange]  reihe_fertig == False
│   │   │   ├─ [Aufruf]  KanteSuchen
│   │   │   ├─ [If]  reihe_fertig == False
│   │   │   │   ├─ [If]  fehler_code != 0
│   │   │   │   │   └─ [Popup: anhalten]  "Kante liegt falsch (Code 9). Breiten prüfen."
│   │   │   │   ├─ [If]  schiene_nr >= anzahl_schienen
│   │   │   │   │   └─ [Popup: anhalten]  "Mehr Reihen als Schienen."
│   │   │   │   ├─ [Aufruf]  TypErkennen
│   │   │   │   ├─ [If]  fehler_code != 0
│   │   │   │   │   └─ [Popup: anhalten]  "Typ nicht erkannt (Code 7) oder Kraft zu hoch (Code 5)."
│   │   │   │   │
│   │   │   │   ├─ [Zuweisung]  versuche_entnehmen = 0
│   │   │   │   ├─ [Zuweisung]  fehler_code = 1
│   │   │   │   ├─ [Schleife solange]  fehler_code != 0 and fehler_code != 5 and fehler_code != 7 and versuche_entnehmen < 3
│   │   │   │   │   ├─ [Aufruf]  Entnehmen
│   │   │   │   │   └─ [Zuweisung]  versuche_entnehmen = versuche_entnehmen + 1
│   │   │   │   ├─ [If]  fehler_code != 0
│   │   │   │   │   └─ [Popup: anhalten]  "Entnehmen fehlgeschlagen. fehler_code im Reiter Variables ansehen."
│   │   │   │   │
│   │   │   │   ├─ [Zuweisung]  ablage_x = schienen_ende_x[schiene_nr] - schienen_belegt[schiene_nr] - breite[typ_nr] / 2
│   │   │   │   ├─ [Zuweisung]  ablage_y = schienen_y[schiene_nr] + nut_versatz[typ_nr]
│   │   │   │   ├─ [Aufruf]  Einsetzen
│   │   │   │   ├─ [If]  fehler_code != 0
│   │   │   │   │   └─ [Popup: anhalten]  "Einsetzen fehlgeschlagen. fehler_code im Reiter Variables ansehen."
│   │   │   │   │
│   │   │   │   ├─ [Script]  schienen_belegt[schiene_nr] = schienen_belegt[schiene_nr] + breite[typ_nr] + abstand_teile
│   │   │   │   ├─ [Zuweisung]  kante_erwartet = kante_x - breite[typ_nr]
│   │   │   │   ├─ [Zuweisung]  erstes_teil = False
│   │   │   │   └─ [Zuweisung]  teile_in_reihe = teile_in_reihe + 1
│   │   │
│   │   │   # --- H: War die Reihe leer? Dann ist die Platte fertig ---
│   │   ├─ [If]  teile_in_reihe == 0
│   │   │   └─ [Zuweisung]  weitere_reihen = False
│   │   ├─ [Else]
│   │   │   └─ [Zuweisung]  reihen_gefunden = reihen_gefunden + 1
│   │   └─ [Zuweisung]  reihe = reihe + 1
│   │
│   ├─ [MoveJ]
│   │   └─ [Wegpunkt fest]  Home
│   └─ [Script]  popup(str_cat("Fertig. Gefundene Reihen: ", to_str(reihen_gefunden)), "Pick and Place", False, False, blocking=True)
│
├─ [Unterprogramm]  KanteSuchen                          # --- I ---
│   ├─ [Zuweisung]  fehler_code = 0
│   ├─ [If]  erstes_teil
│   │   ├─ [Zuweisung]  such_start_x = start_x_rechts
│   │   └─ [Zuweisung]  such_weg = suchweg_erstes_teil
│   ├─ [Else]
│   │   ├─ [Zuweisung]  such_start_x = kante_erwartet + 0.010 + halbe_fingerbreite
│   │   └─ [Zuweisung]  such_weg = 0.025
│   ├─ [Script]  rq_close_and_wait()
│   ├─ [Zuweisung]  ziel = pose_trans(Platte, p[such_start_x, teil_y, tast_hoehe - sicher_hoehe, 0, 0, greifer_drehung])
│   ├─ [MoveJ]
│   │   └─ [Wegpunkt Variable]  ziel
│   ├─ [Zuweisung]  ziel = pose_trans(Platte, p[such_start_x, teil_y, tast_hoehe, 0, 0, greifer_drehung])
│   ├─ [MoveL  50 mm/s]
│   │   └─ [Wegpunkt Variable]  ziel
│   ├─ [Script]  kontakt_x = kante_antasten(Platte, -1, such_weg)
│   ├─ [Zuweisung]  ziel = pose_trans(Platte, p[x_im_frame(Platte) + 0.002, teil_y, tast_hoehe, 0, 0, greifer_drehung])
│   ├─ [MoveL  20 mm/s]
│   │   └─ [Wegpunkt Variable]  ziel
│   ├─ [Zuweisung]  ziel = pose_trans(Platte, p[x_im_frame(Platte), teil_y, tast_hoehe - sicher_hoehe, 0, 0, greifer_drehung])
│   ├─ [MoveL  50 mm/s]
│   │   └─ [Wegpunkt Variable]  ziel
│   ├─ [If]  kontakt_x < -5
│   │   └─ [Zuweisung]  reihe_fertig = True
│   └─ [Else]
│       ├─ [Zuweisung]  kante_x = kontakt_x - halbe_fingerbreite
│       └─ [If]  erstes_teil == False and norm(kante_x - kante_erwartet) > 0.003
│           └─ [Zuweisung]  fehler_code = 9
│
├─ [Unterprogramm]  TypErkennen                          # --- J ---
│   ├─ [Zuweisung]  fehler_code = 0
│   ├─ [Zuweisung]  typ_test = typ_nr
│   ├─ [Zuweisung]  suche_status = 0
│   ├─ [Zuweisung]  test_x = kante_x - kleinste_breite / 2
│   ├─ [Zuweisung]  ziel = pose_trans(Platte, p[test_x, teil_y, greif_tiefe[typ_test] - sicher_hoehe, 0, 0, greifer_drehung])
│   ├─ [MoveJ]
│   │   └─ [Wegpunkt Variable]  ziel
│   ├─ [Schleife solange]  suche_status == 0
│   │   ├─ [Zuweisung]  ziel = pose_trans(Platte, p[test_x, teil_y, greif_tiefe[typ_test] - sicher_hoehe, 0, 0, greifer_drehung])
│   │   ├─ [Script]  rq_move_and_wait(finger_pos[typ_test] - extra_oeffnung)
│   │   ├─ [Script]  fehler_code = kraft_fahren_z(Platte, 1, 5.0, 5.0, 5.0, greif_tiefe[typ_test], 0.001, 5.0)
│   │   ├─ [If]  fehler_code == 0
│   │   │   └─ [Zuweisung]  suche_status = 1
│   │   ├─ [ElseIf]  fehler_code == 5
│   │   │   └─ [Zuweisung]  suche_status = 3
│   │   └─ [Else]
│   │       ├─ [MoveL  20 mm/s]
│   │       │   └─ [Wegpunkt Variable]  ziel
│   │       ├─ [Zuweisung]  typ_test = typ_test + 1
│   │       └─ [If]  typ_test >= anzahl_typen
│   │           └─ [Zuweisung]  suche_status = 2
│   │
│   ├─ [If]  suche_status == 1
│   │   ├─ [Script]  rq_set_force(50)
│   │   ├─ [Script]  rq_close_and_wait()
│   │   ├─ [Script]  typ_gemessen = typ_finden(finger_pos, rq_current_pos(), finger_toleranz)
│   │   ├─ [Script]  rq_move_and_wait(finger_pos[typ_test] - extra_oeffnung)
│   │   ├─ [Zuweisung]  ziel = pose_trans(Platte, p[test_x, teil_y, greif_tiefe[typ_test] - sicher_hoehe, 0, 0, greifer_drehung])
│   │   ├─ [MoveL  50 mm/s]
│   │   │   └─ [Wegpunkt Variable]  ziel
│   │   ├─ [If]  typ_gemessen < 0
│   │   │   └─ [Zuweisung]  fehler_code = 7
│   │   └─ [Else]
│   │       ├─ [Zuweisung]  typ_nr = typ_gemessen
│   │       └─ [Zuweisung]  teil_x = kante_x - breite[typ_nr] / 2
│   ├─ [ElseIf]  suche_status == 2
│   │   ├─ [Zuweisung]  ziel = pose_trans(Platte, p[test_x, teil_y, greif_tiefe[0] - sicher_hoehe, 0, 0, greifer_drehung])
│   │   ├─ [MoveL  50 mm/s]
│   │   │   └─ [Wegpunkt Variable]  ziel
│   │   └─ [Zuweisung]  fehler_code = 7
│   └─ [ElseIf]  suche_status == 3
│       └─ [Zuweisung]  fehler_code = 5
│
├─ [Unterprogramm]  Entnehmen                            # --- K ---
│   ├─ [Script]  rq_move_and_wait(finger_pos[typ_nr] - extra_oeffnung)
│   ├─ [Zuweisung]  ziel = pose_trans(Platte, p[teil_x, teil_y, greif_tiefe[typ_nr] - sicher_hoehe, 0, 0, greifer_drehung])
│   ├─ [MoveJ]
│   │   └─ [Wegpunkt Variable]  ziel
│   ├─ [Zuweisung]  ziel = pose_trans(Platte, p[teil_x, teil_y, greif_tiefe[typ_nr], 0, 0, greifer_drehung])
│   ├─ [MoveL  50 mm/s]
│   │   └─ [Wegpunkt Variable]  ziel
│   │   # Selbstzentrierung: Arm nur in y weich, während der Greifer schließt
│   ├─ [Script]  zero_ftsensor()
│   ├─ [Script]  force_mode(Platte, [0, 1, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0], 2, [0.02, 0.01, 0.02, 0.1, 0.1, 0.1])
│   ├─ [Script]  rq_set_force(greifkraft[typ_nr])
│   ├─ [Script]  rq_close_and_wait()
│   ├─ [Script]  end_force_mode()
│   ├─ [Zuweisung]  fehler_code = 0
│   ├─ [If]  not rq_is_object_detected()
│   │   └─ [Zuweisung]  fehler_code = 1
│   ├─ [ElseIf]  norm(rq_current_pos() - finger_pos[typ_nr]) > finger_toleranz
│   │   └─ [Zuweisung]  fehler_code = 7
│   ├─ [If]  fehler_code == 0
│   │   └─ [Script]  fehler_code = kraft_fahren_z(Platte, -1, 5.0, zieh_kraft[typ_nr], 5.0, greif_tiefe[typ_nr] - zieh_weg[typ_nr], 0.001, 0.6)
│   ├─ [If]  fehler_code != 0
│   │   └─ [Script]  rq_move_and_wait(finger_pos[typ_nr] - extra_oeffnung)
│   ├─ [If]  fehler_code != 5
│   │   ├─ [Zuweisung]  ziel = pose_trans(Platte, p[teil_x, teil_y, greif_tiefe[typ_nr] - sicher_hoehe, 0, 0, greifer_drehung])
│   │   └─ [MoveL  50 mm/s]
│   │       └─ [Wegpunkt Variable]  ziel
│   └─ [If]  fehler_code == 0 and (not rq_is_object_detected() or norm(rq_current_pos() - finger_pos[typ_nr]) > finger_toleranz)
│       └─ [Zuweisung]  fehler_code = 3
│
└─ [Unterprogramm]  Einsetzen                            # --- L ---
    ├─ [MoveJ]
    │   └─ [Wegpunkt fest]  Zwischen
    ├─ [Zuweisung]  fehler_code = 0
    ├─ [Zuweisung]  einsetz_status = 0
    ├─ [If]  not rq_is_object_detected()
    │   └─ [Zuweisung]  fehler_code = 3
    ├─ [If]  fehler_code == 0
    │   ├─ [Zuweisung]  ziel = pose_trans(Schrank, p[ablage_x, ablage_y, einrast_tiefe[typ_nr] - sicher_hoehe, 0, 0, greifer_drehung])
    │   ├─ [MoveJ]
    │   │   └─ [Wegpunkt Variable]  ziel
    │   ├─ [Zuweisung]  versuch_einsetzen = 0
    │   ├─ [Schleife solange]  versuch_einsetzen < 3 and einsetz_status == 0
    │   │   ├─ [Zuweisung]  ziel = pose_trans(Schrank, p[ablage_x + versatz_x[versuch_einsetzen], ablage_y, einrast_tiefe[typ_nr] - 0.015, 0, 0, greifer_drehung])
    │   │   ├─ [MoveL  50 mm/s]
    │   │   │   └─ [Wegpunkt Variable]  ziel
    │   │   ├─ [Script]  fehler_code = kraft_fahren_z(Schrank, 1, 5.0, druck_kraft[typ_nr], 5.0, einrast_tiefe[typ_nr], 0.001, 0.6)
    │   │   ├─ [If]  fehler_code == 5
    │   │   │   └─ [Zuweisung]  einsetz_status = 2
    │   │   └─ [Else]
    │   │       ├─ [Zuweisung]  sitzt_fest = False
    │   │       ├─ [If]  fehler_code == 0
    │   │       │   └─ [Script]  sitzt_fest = sitzt_teil_fest(Schrank, einrast_tiefe[typ_nr], 10.0)
    │   │       ├─ [If]  sitzt_fest
    │   │       │   ├─ [Script]  rq_move_and_wait(finger_pos[typ_nr] - extra_oeffnung)
    │   │       │   ├─ [Zuweisung]  ziel = pose_trans(Schrank, p[ablage_x, ablage_y, einrast_tiefe[typ_nr] - sicher_hoehe, 0, 0, greifer_drehung])
    │   │       │   ├─ [MoveL  50 mm/s]
    │   │       │   │   └─ [Wegpunkt Variable]  ziel
    │   │       │   └─ [Zuweisung]  einsetz_status = 1
    │   │       └─ [Else]
    │   │           ├─ [Zuweisung]  ziel = pose_trans(Schrank, p[ablage_x, ablage_y, einrast_tiefe[typ_nr] - 0.015, 0, 0, greifer_drehung])
    │   │           ├─ [MoveL  50 mm/s]
    │   │           │   └─ [Wegpunkt Variable]  ziel
    │   │           └─ [Zuweisung]  versuch_einsetzen = versuch_einsetzen + 1
    │   ├─ [If]  einsetz_status == 0
    │   │   └─ [Zuweisung]  fehler_code = 4
    │   └─ [If]  einsetz_status == 1
    │       └─ [Zuweisung]  fehler_code = 0
```

**Einstellungen in PolyScope / تنظیمات در PolyScope:**
- **Feature in jedem Move-Knoten = Base**, weil `ziel` schon mit `pose_trans` umgerechnet ist. / در همهٔ گره‌های Move، Feature = Base.
- **Check Expression Continuously** bei allen Schleifen und If **aus**. / خاموش.
- **Blend = 0**.
- Popups: ☑ **Halt program execution at this popup**. Danach **Stop** drücken, nicht Play. / بعد از پیام خطا **Stop**، نه Play.

---

## 2. Erklärung der Variablen / توضیح متغیرها

Einheiten: Länge in **Meter**, Kraft in **Newton**, Finger **0–255** (0 = offen, 255 = zu). Negatives z = **über** der Fläche.
واحدها: طول به **متر**، نیرو به **نیوتن**، انگشت **۰ تا ۲۵۵** (۰ = باز، ۲۵۵ = بسته). z منفی = **بالای** سطح.

### A: Teiletypen (eine Spalte pro Typ) / انواع کلید (هر ستون یک نوع)

| Variable | Deutsch | فارسی |
|---|---|---|
| `breite` | Breite des Teils entlang der Reihe (x) | عرض کلید در امتداد ردیف (x) |
| `greifkraft` | Wie fest der Greifer zudrückt (0–255) | گریپر چقدر محکم فشار بدهد |
| `greif_tiefe` | So tief fährt der Greifer auf der Platte zum Greifen | گریپر برای گرفتن تا کجا روی برد پایین برود |
| `einrast_tiefe` | So tief sitzt das Teil, wenn es auf der Schiene eingerastet ist | کلید وقتی روی ریل جا افتاد در چه عمقی است |
| `zieh_kraft` | Höchste Kraft beim Herausziehen | حداکثر نیرو برای بیرون کشیدن |
| `druck_kraft` | Höchste Kraft beim Aufdrücken auf die Schiene | حداکثر نیرو برای فشار دادن روی ریل |
| `zieh_weg` | So weit wird das Teil hochgezogen, bis es frei ist | کلید تا کجا بالا کشیده شود تا آزاد شود |
| `finger_pos` | Fingerstellung, wenn der Greifer dieses Teil hält. **Daran erkennt der Roboter den Typ** | موقعیت انگشت وقتی این کلید را گرفته. **ربات با این عدد نوع را می‌شناسد** |
| `nut_versatz` | Wie weit die Nut von der Teilmitte entfernt ist (Lochplatte) | شیار چقدر از وسط کلید فاصله دارد (برد سوراخ‌دار) |
| `anzahl_typen` | Wie viele Typen es gibt (wird gezählt) | چند نوع هست (شمرده می‌شود) |

### B: Greifer und Tasten / گریپر و لمس

| Variable | Deutsch | فارسی |
|---|---|---|
| `kleinste_breite` | Breite des schmalsten Teils. Beim Typ-Test setzt der Roboter so weit neben der Kante an, dass er sicher auf dem Teil ist | عرض باریک‌ترین کلید. ربات برای امتحان نوع آنقدر از لبه دور می‌شود که حتماً روی کلید باشد |
| `finger_toleranz` | Erlaubte Abweichung bei der Fingerstellung | اختلاف مجاز در موقعیت انگشت |
| `extra_oeffnung` | Greifer so viel weiter öffnen als das Teil, damit er nicht anstößt | گریپر کمی بیشتر از کلید باز شود تا به آن نخورد |
| `halbe_fingerbreite` | Halbe Breite der geschlossenen Finger, für die genaue Kante | نصف پهنای انگشت‌های بسته، برای جای دقیق لبه |
| `tast_hoehe` | Auf dieser Höhe sucht der Roboter die Kante | ربات در این ارتفاع دنبال لبه می‌گردد |
| `sicher_hoehe` | So hoch über dem Teil fährt der Roboter sicher hin und her (6 cm) | ارتفاع امن بالای کلید برای جابه‌جا شدن (۶ سانتی‌متر) |
| `greifer_drehung` | 0 oder 1.5708 (90°), damit die Finger die beiden Enden der Länge greifen | ۰ یا ۹۰ درجه، تا انگشت‌ها دو سر طول کلید را بگیرند |

### C: Platte / برد

| Variable | Deutsch | فارسی |
|---|---|---|
| `reihen_y` | Lage der bekannten Reihen (Reihe 1, Reihe 2) | جای ردیف‌های شناخته‌شده |
| `reihen_abstand` | Abstand der Reihen, um weitere Reihen zu suchen | فاصلهٔ ردیف‌ها، برای پیدا کردن ردیف‌های بیشتر |
| `start_x_rechts` | Startpunkt rechts neben den Reihen, im freien Bereich | نقطهٔ شروع سمت راست ردیف‌ها، در جای خالی |
| `suchweg_erstes_teil` | So weit sucht der Roboter nach links nach dem ersten Teil | ربات تا این فاصله به چپ دنبال اولین کلید می‌گردد |
| `max_reihen` | Höchstens so viele Reihen testen | حداکثر چند ردیف امتحان شود |

### D: Schrank / کمد

| Variable | Deutsch | فارسی |
|---|---|---|
| `schienen_y` | Mitte jeder Schiene. Reihe 1 → erste Zahl, Reihe 2 → zweite Zahl | وسط هر ریل. ردیف ۱ ← عدد اول، ردیف ۲ ← عدد دوم |
| `schienen_ende_x` | Rechtes Ende jeder Schiene, dort kommt das erste Teil hin | انتهای راست هر ریل، اولین کلید آنجا می‌رود |
| `schienen_belegt` | Wie viel jeder Schiene schon voll ist | هر ریل چقدر پر شده |
| `anzahl_schienen` | Wie viele Schienen es gibt | چند ریل هست |
| `abstand_teile` | Lücke zwischen den Teilen auf der Schiene (0 = dicht) | فاصلهٔ بین کلیدها روی ریل (۰ = چسبیده) |
| `versatz_x` | Verschiebung bei Versuch 1, 2, 3 beim Einsetzen | جابه‌جایی در تلاش ۱، ۲ و ۳ هنگام جا انداختن |

### E: Arbeitsvariablen (der Schmierzettel des Roboters) / متغیرهای کاری (چرک‌نویس ربات)

| Variable | Deutsch | فارسی |
|---|---|---|
| `reihe` | Nummer der aktuellen Reihe (0, 1, 2 …) | شمارهٔ ردیف فعلی |
| `weitere_reihen` | True = vielleicht gibt es noch eine Reihe | True = شاید ردیف دیگری هم باشد |
| `reihen_gefunden` | Wie viele Reihen bearbeitet wurden | چند ردیف انجام شد |
| `teile_in_reihe` | Wie viele Teile aus dieser Reihe genommen wurden | چند کلید از این ردیف برداشته شد |
| `reihe_fertig` | True = in dieser Reihe ist kein Teil mehr | True = در این ردیف کلیدی نمانده |
| `erstes_teil` | True = das erste Teil der Reihe wird gesucht | True = دنبال اولین کلید ردیف است |
| `typ_nr` | Typ des aktuellen Teils | نوع کلید فعلی |
| `typ_test` | Typ, der gerade ausprobiert wird | نوعی که الان امتحان می‌شود |
| `typ_gemessen` | Typ, der aus der Fingerstellung erkannt wurde | نوعی که از موقعیت انگشت شناخته شد |
| `suche_status` | 0 sucht noch, 1 passt, 2 kein Typ passt, 3 Kraft zu hoch | ۰ هنوز می‌گردد، ۱ جور شد، ۲ هیچ نوعی جور نشد، ۳ نیرو زیاد |
| `kontakt_x` | Wo der Finger das Teil berührt hat | جایی که انگشت به کلید خورد |
| `kante_x` | Rechte Kante des Teils | لبهٔ راست کلید |
| `kante_erwartet` | Wo die Kante des nächsten Teils sein müsste | لبهٔ کلید بعدی باید کجا باشد |
| `such_start_x`, `such_weg` | Wo die Suche beginnt und wie weit sie geht | جستجو از کجا شروع شود و تا کجا برود |
| `test_x` | Punkt für den Typ-Test | نقطهٔ امتحان نوع |
| `teil_x`, `teil_y` | Mitte des Teils auf der Platte | وسط کلید روی برد |
| `schiene_nr` | Auf welche Schiene das Teil kommt | کلید روی کدام ریل برود |
| `ablage_x`, `ablage_y` | Wo genau das Teil auf der Schiene hinkommt | کلید دقیقاً کجای ریل گذاشته شود |
| `ziel` | Punkt, zu dem der Roboter gleich fährt | نقطه‌ای که ربات الان به آن می‌رود |
| `fehler_code` | 0 = alles gut, sonst Fehlerart (Abschnitt 4) | ۰ = همه‌چیز خوب، وگرنه نوع خطا |
| `versuche_entnehmen` | Versuche beim Herausnehmen (höchstens 3) | تعداد تلاش برای برداشتن (حداکثر ۳) |
| `versuch_einsetzen` | Versuche beim Einsetzen (höchstens 3) | تعداد تلاش برای جا انداختن (حداکثر ۳) |
| `einsetz_status` | 0 noch nicht, 1 eingerastet, 2 Kraft zu hoch | ۰ هنوز نه، ۱ جا افتاد، ۲ نیرو زیاد |
| `sitzt_fest` | True = Zugtest bestanden | True = تست کشش موفق |

---

## 3. Erklärung jedes Programmteils / توضیح هر بخش برنامه

### BeforeStart: Vorbereitung / آماده‌سازی
- **DE:** Läuft einmal beim Start. Die Funktionsdatei wird geladen, der Greifer wird zurückgesetzt und aktiviert. Danach bekommt jede Variable ihren Anfangswert (Teile A–E).
- **FA:** یک بار در شروع اجرا می‌شود. فایل تابع‌ها بار می‌شود و گریپر ریست و فعال می‌شود. بعد هر متغیر مقدار اولیه‌اش را می‌گیرد (بخش‌های A تا E).

### F: Reihe für Reihe / ردیف به ردیف
- **DE:** Die äußere Schleife läuft, solange Reihen gefunden werden, höchstens `max_reihen`. Für jede Reihe wird ausgerechnet, wo sie liegt (`teil_y`), und sie bekommt die Schiene mit derselben Nummer (`schiene_nr = reihe`). So kommt Reihe 1 auf Schiene 1, Reihe 2 auf Schiene 2. Die Kantensuche startet ganz rechts.
- **FA:** حلقهٔ بیرونی تا وقتی ردیف پیدا می‌شود ادامه دارد، حداکثر `max_reihen`. برای هر ردیف حساب می‌شود کجاست (`teil_y`) و ریلی با همان شماره به آن داده می‌شود (`schiene_nr = reihe`). پس ردیف ۱ روی ریل ۱ می‌رود و ردیف ۲ روی ریل ۲. جستجوی لبه از سمت راست شروع می‌شود.

### G: Teil für Teil / کلید به کلید
- **DE:** Die innere Schleife holt ein Teil nach dem anderen, von rechts nach links: Kante suchen → Typ erkennen → bis zu 3-mal versuchen zu entnehmen → Platz auf der Schiene ausrechnen → einsetzen. Nach jedem Teil merkt sich der Roboter, wie viel Schiene belegt ist und wo die nächste Kante sein müsste. Bei jedem Fehler kommt ein Popup und das Programm hält an.
- **FA:** حلقهٔ داخلی کلیدها را یکی‌یکی از راست به چپ برمی‌دارد: پیدا کردن لبه ← تشخیص نوع ← تا ۳ بار تلاش برای برداشتن ← حساب کردن جا روی ریل ← جا انداختن. بعد از هر کلید، ربات یادداشت می‌کند چقدر از ریل پر شده و لبهٔ بعدی باید کجا باشد. در هر خطا یک پیام می‌آید و برنامه می‌ایستد.

### H: Reihen zählen / شمردن ردیف‌ها
- **DE:** Wurde in einer Reihe **kein** Teil gefunden, ist die Platte fertig. Sonst wird die Reihe gezählt und die nächste getestet. Am Ende zeigt ein Popup, wie viele Reihen gefunden wurden.
- **FA:** اگر در یک ردیف **هیچ** کلیدی پیدا نشد، برد تمام است. وگرنه ردیف شمرده می‌شود و ردیف بعدی امتحان می‌شود. در آخر یک پیام می‌گوید چند ردیف پیدا شد.

### I: KanteSuchen / پیدا کردن لبه
- **DE:** Greifer zu. Beim ersten Teil startet der Roboter rechts im freien Bereich, sonst 1 cm rechts von der erwarteten Kante. Er fährt nach unten auf `tast_hoehe` und schiebt dann **mit 8 N langsam nach links**, bis er anstößt. Dann 2 mm zurück und hoch. Kontakt minus halbe Fingerbreite = rechte Kante des Teils. Findet er nichts, ist die Reihe fertig. Liegt die Kante mehr als 3 mm falsch, gibt es Code 9.
- **FA:** گریپر بسته. برای اولین کلید ربات از سمت راست در جای خالی شروع می‌کند، وگرنه ۱ سانتی‌متر سمت راست لبهٔ مورد انتظار. تا `tast_hoehe` پایین می‌رود و بعد **با ۸ نیوتن آرام به چپ** هل می‌دهد تا به کلید بخورد. بعد ۲ میلی‌متر برمی‌گردد و بالا می‌رود. جای تماس منهای نصف انگشت = لبهٔ راست کلید. اگر چیزی پیدا نکند، ردیف تمام است. اگر لبه بیش از ۳ میلی‌متر جابه‌جا باشد، کد ۹.

### J: TypErkennen / تشخیص نوع
- **DE:** Der Roboter öffnet den Greifer für den kürzesten möglichen Typ und fährt **mit nur 5 N** nach unten. Setzen die Finger auf dem Teil auf, ist das Teil länger: hoch, weiter öffnen, nächster Typ. Passen die Finger daneben, schließt er, liest die Fingerstellung und sucht in `finger_pos` den passenden Typ. Dann rechnet er die Teilmitte aus.
- **FA:** ربات گریپر را به اندازهٔ کوتاه‌ترین نوع ممکن باز می‌کند و **فقط با ۵ نیوتن** پایین می‌رود. اگر انگشت‌ها روی کلید بنشینند، کلید بلندتر است: بالا، بیشتر باز، نوع بعدی. اگر انگشت‌ها کنار کلید جا شوند، می‌بندد، موقعیت انگشت را می‌خواند و در `finger_pos` نوع مناسب را پیدا می‌کند. بعد وسط کلید را حساب می‌کند.

### K: Entnehmen / برداشتن
- **DE:** Greifer etwas weiter als das Teil öffnen, über die Teilmitte, herunter. Beim Schließen ist der Arm **nur in y weich**: Berührt ein Finger zuerst, gibt der Arm nach, bis beide anliegen. So wird das Teil **in der Mitte** gegriffen. Dann prüfen: Ist etwas im Greifer? Passt die Fingerstellung? Danach **kraftgeregelt** hochziehen, höchstens mit `zieh_kraft`. Zum Schluss nochmal prüfen, ob das Teil noch im Greifer ist (sonst Code 3).
- **FA:** گریپر کمی بیشتر از کلید باز، بالای وسط کلید، پایین. هنگام بستن، بازو **فقط در y نرم** است: اگر یک انگشت زودتر بخورد، بازو کنار می‌رود تا هر دو بچسبند. پس کلید **از وسط** گرفته می‌شود. بعد چک: چیزی در گریپر هست؟ موقعیت انگشت جور است؟ بعد **با کنترل نیرو** بالا می‌کشد، حداکثر با `zieh_kraft`. در آخر دوباره چک می‌کند کلید هنوز در گریپر هست (وگرنه کد ۳).

### L: Einsetzen / جا انداختن
- **DE:** Über den Zwischenpunkt zum Schrank. Über die Ablagestelle, auf 15 mm herunter, dann **mit steigender Kraft** auf die Schiene drücken. Danach **Zugtest**: mit 10 N nach oben ziehen. Bleibt das Teil unten, sitzt es fest → Greifer auf, hoch. Sonst noch einmal mit 2 mm Versatz nach rechts, dann nach links. Nach 3 Fehlversuchen Code 4.
- **FA:** از نقطهٔ میانی به کمد. بالای جای گذاشتن، تا ۱۵ میلی‌متر پایین، بعد **با نیروی رو به افزایش** روی ریل فشار می‌دهد. بعد **تست کشش**: با ۱۰ نیوتن بالا می‌کشد. اگر کلید پایین بماند، محکم است ← گریپر باز، بالا. وگرنه دوباره با ۲ میلی‌متر جابه‌جایی به راست، بعد به چپ. بعد از ۳ تلاش ناموفق، کد ۴.

### Funktionen in `funktionen_v5_lesbar.script` / تابع‌ها

| Funktion | Deutsch | فارسی |
|---|---|---|
| `x_im_frame`, `y_im_frame`, `z_im_frame` | Wo ist die Werkzeugspitze, gemessen im Feature Platte oder Schrank? | نوک ابزار در مختصات برد یا کمد کجاست؟ |
| `reihen_position` | y-Lage einer Reihe, auch für Reihen, die nicht in der Liste stehen | جای y یک ردیف، حتی ردیفی که در لیست نیست |
| `kraft_fahren_z` | Hoch oder runter fahren mit Kraftregelung, die Kraft steigt in Stufen. Über 100 N sofort Stopp | بالا یا پایین رفتن با کنترل نیرو، نیرو پله‌پله زیاد می‌شود. بالای ۱۰۰ نیوتن فوراً توقف |
| `sitzt_teil_fest` | Zugtest: sitzt das Teil fest auf der Schiene? | تست کشش: کلید محکم روی ریل است؟ |
| `kante_antasten` | Seitlich schieben, bis der Finger stehen bleibt = Kante gefunden | از کنار هل دادن تا انگشت بایستد = لبه پیدا شد |
| `typ_finden` | Fingerstellung mit der Liste vergleichen → Typnummer | مقایسهٔ موقعیت انگشت با لیست ← شمارهٔ نوع |

---

## 4. Fehlercodes / کدهای خطا

| Code | Deutsch | فارسی |
|---|---|---|
| 0 | alles gut | همه‌چیز خوب |
| 1 | nichts gegriffen | چیزی گرفته نشد |
| 2 | Ziel mit Höchstkraft nicht erreicht | با حداکثر نیرو نرسید |
| 3 | Teil verloren oder gerutscht | کلید افتاد یا لیز خورد |
| 4 | nach 3 Versuchen nicht eingerastet | بعد از ۳ تلاش جا نیفتاد |
| 5 | Notgrenze 100 N | از ۱۰۰ نیوتن گذشت |
| 7 | Typ nicht erkannt | نوع شناخته نشد |
| 9 | Kante mehr als 3 mm falsch | لبه بیش از ۳ میلی‌متر جابه‌جا |
