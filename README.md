## PROG101_ASSIGNMENT1_EMMANUEL_ALVIN_MACARTHY_905006533
# Freetown Community Flood Early Alert System (FCFEAS)


---

## 1. The community problem

Freetown floods every rainy season, but the warning that reaches a family in a flood-prone settlement is often late, unclear or absent. The problem addressed here is the **lack of a timely, ward-specific and easily understood flood alert that connects weather and water data to community action**.

- **Who is affected:** residents of low-lying coastal and riverside settlements such as Kroo Bay, Portee and Rokupa, and hillside areas such as Regent. Informal-settlement households, women, children, older people and people with disabilities suffer most.
- **Why it exists:** rapid population growth pushes households onto floodplains and steep slopes, the warning chain is fragmented, and many people cannot be reached by text-heavy or internet-based messages.
- **How FCFEAS helps:** it collects rainfall, water-level and community readings, classifies every ward into one of four alert levels, sends short SMS alerts in English and Krio, activates volunteers and radio for serious alerts, and escalates automatically if nobody acknowledges.

## 2. How the system works

FCFEAS is a monitoring cycle that repeats every **15 minutes**.

- **Inputs:** 24-hour rainfall (`rain24`), hourly rainfall intensity (`rainHr`), water level as a percentage of channel capacity (`waterPct`), and SMS flood reports from residents (`reportCount`).
- **Processing:** data validation, risk classification per ward, message creation, retries and escalation rules.
- **Outputs:** SMS alerts, radio and volunteer notifications, sirens for the highest level, and an audit log.

### Alert levels

Thresholds are illustrative and must be calibrated with SLMet and NDMA. Rainfall thresholds are multiplied by 0.8 in high-risk wards. "Confirmed" means at least 3 community reports in the cycle.

| Level | Trigger (any one condition) | Public action |
| --- | --- | --- |
| **GREEN** | Below all thresholds | No alert. "All clear" is sent when returning from a higher level. |
| **YELLOW** | rain24 ≥ 50 mm, or water level ≥ 60% | Be alert: clear drains, secure belongings, listen to radio. |
| **ORANGE** | rain24 ≥ 100 mm, or water ≥ 80%, or (rain24 ≥ 50 mm and confirmed reports) | Prepare to move. Volunteers activated. |
| **RED** | rain24 ≥ 150 mm, or water ≥ 100%, or (rainfall ≥ 30 mm/h and confirmed reports) | Move to higher ground now. Siren and door-to-door alert. |

Alerts are sent **only when a ward's level changes**, so residents are not flooded with repeated messages.

# FCFEAS Algorithm Design

**Project:** Freetown Community Flood Early Alert System (FCFEAS)
**Module:** PROG101 Principles of Programming Logic and Design
**Author:** Emmanuel Alvin Macarthy (905006553)

## 1. Purpose

FCFEAS is a logical design (no source code) of a system that repeats a monitoring cycle every 15 minutes. It collects rainfall, water level and resident SMS reports, classifies each ward into one of four alert levels (GREEN, YELLOW, ORANGE, RED), and sends short SMS alerts in English and Krio. For serious alerts it also notifies radio, volunteers and sirens, and it escalates automatically if nobody acknowledges. It addresses the lack of timely, ward-specific flood warnings in Freetown (SDG 11 and SDG 13).

## 2. Input, Processing, Output

| Stage | Details |
|---|---|
| **Input** | rain24 (24-hour rainfall), rainHr (hourly rainfall), waterPct (water level as % of channel capacity), resident SMS (FLOOD, JOIN, STOP) |
| **Processing** | validation, high-risk factor, report confirmation, risk classification, message building, retries, escalation |
| **Output** | SMS alerts, radio and volunteer notifications, sirens (RED only), acknowledgement and escalation calls, log entries |

## 3. Constants

| Constant | Value | Data type | Meaning |
|---|---|---|---|
| CYCLE_MINUTES | 15 | Integer | Time between cycles |
| RAIN_YELLOW, RAIN_ORANGE, RAIN_RED | 50, 100, 150 | Real (mm) | 24-hour rainfall thresholds |
| INTENSITY_RED | 30 | Real (mm/h) | Hourly rainfall for RED when confirmed |
| WATER_YELLOW, WATER_ORANGE, WATER_RED | 60, 80, 100 | Real (%) | Water level thresholds |
| CONFIRM_COUNT | 3 | Integer | Reports needed to confirm flooding |
| HIGH_RISK_FACTOR | 0.8 | Real | Lowers rainfall thresholds in high-risk wards |
| ACK_WAIT_MINUTES | 10 | Integer | Wait for acknowledgement before escalation |
| MAX_RETRIES | 3 | Integer | SMS retry attempts |
| RAIN24_MAX, RAINHR_MAX, WATER_MAX | 500, 200, 300 | Real | Sanity limits for validation |

Values are illustrative and must be calibrated with SLMet and NDMA.

## 4. Main Variables

| Variable | Data type | Purpose |
|---|---|---|
| ward (code, displayName, highRisk) | Record: String, String, Boolean | A monitored ward |
| rain24, rainHr, waterPct | Real | Rainfall and water level readings |
| reportCount | Integer | Resident flood reports in the cycle |
| level, previousLevel | String (GREEN, YELLOW, ORANGE, RED) | Current and last alert level |
| confirmed, acknowledged, systemRunning | Boolean | Report confirmation, acknowledgement, loop control |
| factor | Real | 0.8 for high-risk wards, otherwise 1.0 |
| phone, language | String | Subscriber details (EN or KR) |
| sentCount, failCount, attempts | Integer | SMS delivery counters |
| wards, subscribers, retryQueue | List | Collections processed by loops |

## 5. Step-by-Step Algorithm

1. START the system.
2. Load constants, the ward list (with high-risk flag) and the subscriber list.
3. For every ward, set previousLevel to GREEN. Set systemRunning to TRUE.
4. WHILE systemRunning is TRUE, repeat steps 5 to 19.
5. INPUT: collect rain24 and rainHr, waterPct and incoming SMS. Count FLOOD reports per ward as reportCount.
6. Validate all readings against the sanity limits. If any value is invalid, log the error, flag the duty officer and go to step 19.
7. FOR EACH ward in the ward list, repeat steps 8 to 17.
8. If the ward is high-risk, set factor to 0.8; otherwise set factor to 1.0.
9. Set confirmed to TRUE if reportCount is at least 3.
10. Decision cascade: if the RED condition is met, level is RED; else if the ORANGE condition is met, level is ORANGE; else if the YELLOW condition is met, level is YELLOW; otherwise level is GREEN.
11. If level equals previousLevel, go to step 17 (no new alert).
12. Build the alert message in English and Krio for the new level (GREEN produces an "all clear").
13. Loop: for every subscriber in the ward, choose the language, send the SMS, count successes and queue failures. Retry queued messages up to 3 times.
14. If level is ORANGE or RED, notify community radio, ward volunteers and the NDMA focal point. If RED, also trigger the siren and door-to-door alert.
15. If level is ORANGE or RED, wait 10 minutes for acknowledgement. If none arrives, escalate to the District Disaster Management Committee and the NDMA duty officer.
16. Set previousLevel to level.
17. Move to the next ward (end of FOR EACH loop).
18. Save the cycle results to the log and update the dashboard.
19. WAIT 15 minutes. If a stop signal was received, set systemRunning to FALSE.
20. When the loop ends, save the final report and END.

## 6. Control Structures Used

| Structure | Where it appears |
|---|---|
| Repetition: WHILE | Main cycle (steps 4 and 19); retry loop (step 13) |
| Repetition: FOR EACH | Ward loop (step 7); subscriber loop (step 13); SMS intake |
| Decision: IF / ELSE | Validation (6), high-risk factor (8), level change (11), language choice (13), escalation (15) |
| Decision cascade | RED, ORANGE, YELLOW, GREEN (step 10) |
| Functions | Each task is a function, grouped into modules M1 to M7 |

## 7. Boolean Logic

| Expression | Result when TRUE |
|---|---|
| confirmed = (reportCount >= CONFIRM_COUNT) | Community reports support the sensor data |
| rain24 >= RAIN_RED * factor OR waterPct >= WATER_RED OR (rainHr >= INTENSITY_RED AND confirmed) | Level is RED |
| rain24 >= RAIN_ORANGE * factor OR waterPct >= WATER_ORANGE OR (rain24 >= RAIN_YELLOW * factor AND confirmed) | Level is ORANGE |
| rain24 >= RAIN_YELLOW * factor OR waterPct >= WATER_YELLOW | Level is YELLOW |
| level <> previousLevel | An alert must be sent |
| level = "ORANGE" OR level = "RED" | Notify radio and volunteers, check escalation |
| NOT acknowledged | Escalate to district committee and NDMA duty officer |

## 8. Functional Modules

| Module | Purpose | Functions |
|---|---|---|
| M1 Data Collection | Gather and validate readings and SMS | collectData, receiveSMSReport, isValidData |
| M2 Subscriber Management | Register and remove residents | registerSubscriber, removeSubscriber, getSubscribers |
| M3 Risk Analysis | Classify each ward | calculateRiskLevel, isHighRiskWard |
| M4 Message Builder | Create English and Krio alert text | buildMessage |
| M5 Alert Dissemination | Deliver alerts on all channels | sendAlerts, notifyRadioAndVolunteers, triggerSiren |
| M6 Escalation | Make sure serious alerts are acted on | checkEscalation |
| M7 Logging and Reporting | Keep an audit trail and final report | logEvent, saveFinalReport |

## 9. Sample Trace (Scenario 4: Portee)

**Inputs:** rain24 = 85, rainHr = 10, waterPct = 70, reportCount = 4, previousLevel = YELLOW. PORTEE is a high-risk ward.

| Step | Action | Result |
|---|---|---|
| 8 | Ward is high-risk | factor = 0.8 |
| 9 | 4 >= 3 | confirmed = TRUE |
| 10 | RED test: 85 >= 150 x 0.8 = 120? No. 70 >= 100? No. 10 >= 30? No. | RED not met |
| 10 | ORANGE test: 85 >= 100 x 0.8 = 80? Yes. | level = ORANGE |
| 11 | ORANGE <> YELLOW | Level changed, alert needed |
| 12 to 13 | Build message and send SMS to PORTEE subscribers | English or Krio per subscriber |
| 14 | Level is ORANGE | Radio, volunteers and NDMA focal point notified. No siren. |
| 15 | Wait 10 minutes for acknowledgement | Escalate if none received |
| 16 | previousLevel = ORANGE | Stored for the next cycle |
##  Flowcharts

Two flowcharts show the whole design. Each step is tagged with the module (M1 to M7) that performs it. They are drawn below with Mermaid so they display directly on GitHub, and the draw.io versions are also in this repository as PNG images (`Flowchart_1_Main_Monitoring_Cycle.png` and `Flowchart_2_Alert_Dissemination_and_Escalation.png`).

### Flowchart 1: Main monitoring cycle, ward processing and risk classification

The WHILE loop that runs every 15 minutes, the validation decision, the FOR EACH loop over wards, and the RED, ORANGE, YELLOW or GREEN decision cascade.

<img width="661" height="887" alt="image" src="https://github.com/user-attachments/assets/048051c8-d75a-43a4-a5d6-a06db533cc2c" />


### Flowchart 2: Alert dissemination and escalation

Called from Flowchart 1 when a ward's level changes: the loop over subscribers, English or Krio selection, SMS retries, notification of radio and volunteers, and escalation if nobody acknowledges within 10 minutes.

<img width="667" height="815" alt="image" src="https://github.com/user-attachments/assets/164e255e-9e5e-476e-945d-b8578735e79e" />


## Pseudocode

The complete pseudocode for the main program and all seven modules is in **[pseudocode.md](pseudocode.md)**. The main program is shown here:

```text
PROGRAM FreetownFloodAlertSystem
   // ----- Constants -----
   CONSTANT CYCLE_MINUTES = 15
   CONSTANT RAIN_YELLOW = 50, RAIN_ORANGE = 100, RAIN_RED = 150   // mm in 24 hours
   CONSTANT INTENSITY_RED = 30                                    // mm per hour
   CONSTANT WATER_YELLOW = 60, WATER_ORANGE = 80, WATER_RED = 100 // % of channel
   CONSTANT CONFIRM_COUNT = 3
   CONSTANT HIGH_RISK_FACTOR = 0.8
   CONSTANT ACK_WAIT_MINUTES = 10, MAX_RETRIES = 3
   CONSTANT RAIN24_MAX = 500, RAINHR_MAX = 200, WATER_MAX = 300
   // ----- Variables -----
   DECLARE wards AS ARRAY OF Ward
   DECLARE data AS ARRAY OF WardData
   DECLARE previousLevel AS ARRAY OF STRING
   DECLARE level AS STRING
   DECLARE pair AS MessagePair                 // pair.english and pair.krio
   DECLARE systemRunning AS BOOLEAN

   CALL loadWards(wards)
   FOR EACH ward IN wards DO
      SET previousLevel[ward.code] = "GREEN"
   END FOR
   SET systemRunning = TRUE

   WHILE systemRunning = TRUE DO
      SET data = collectData(wards)
      IF isValidData(data) = FALSE THEN
         CALL logEvent("INVALID DATA", "Duty officer review needed")
      ELSE
         FOR EACH ward IN wards DO
            SET level = calculateRiskLevel(ward, data[ward.code])
            IF level <> previousLevel[ward.code] THEN
               SET pair = buildMessage(ward, level)
               CALL sendAlerts(ward, level, pair)
               CALL checkEscalation(ward, level)
               SET previousLevel[ward.code] = level
            END IF
         END FOR
         CALL logEvent("CYCLE COMPLETE", previousLevel)
      END IF
      WAIT CYCLE_MINUTES minutes
      IF stopSignalReceived() = TRUE THEN SET systemRunning = FALSE
   END WHILE
   CALL saveFinalReport()
END PROGRAM
```

##  Computational thinking

| Technique | Application |
| --- | --- |
| **Decomposition** | The system is split into seven modules (below). |
| **Pattern recognition** | The same wards flood repeatedly and in the same sequence: heavy rain, rising water, resident reports, damage. This justifies thresholds, a high-risk ward list and report confirmation. |
| **Abstraction** | Soil type, drain design and terrain are ignored. Each ward is reduced to four numbers: rain24, rainHr, waterPct and reportCount. |
| **Algorithm design** | An ordered cycle of input, validation, decision cascade, loops over wards and subscribers, and escalation. |

##  Modular design

| Module | Purpose | Functions |
| --- | --- | --- |
| **M1 Data Collection** | Gather and validate readings and SMS reports | `collectData`, `receiveSMSReport`, `isValidData` |
| **M2 Subscriber Management** | Register and remove residents (JOIN and STOP) | `registerSubscriber`, `removeSubscriber`, `getSubscribers` |
| **M3 Risk Analysis** | Classify each ward | `calculateRiskLevel`, `isHighRiskWard` |
| **M4 Message Builder** | Create short alert text in English and Krio | `buildMessage` |
| **M5 Alert Dissemination** | Deliver alerts by SMS, radio, volunteers and siren | `sendAlerts`, `notifyRadioAndVolunteers`, `triggerSiren` |
| **M6 Escalation** | Make sure serious alerts are acted on | `checkEscalation` |
| **M7 Logging and Reporting** | Keep an audit trail and reports | `logEvent`, `saveFinalReport` |

Programming logic concepts used: constants, variables, data types, Boolean logic, decision structures, repetition structures (WHILE and FOR EACH), functions, modular design and problem decomposition.

##  Sample scenarios (expected outputs)

KROO, PORTEE, CULVERT and REGENT are treated as high-risk wards (factor 0.8) and HILLST as a standard ward (factor 1.0). The full set of ten scenarios is in the report.

| Inputs | Result |
| --- | --- |
| KROO: rain24 45 mm, rainHr 8, waterPct 40, reports 0 | 45 ≥ 50 × 0.8 = 40, so the level becomes **YELLOW** and an SMS is sent to KROO subscribers. |
| HILLST: rain24 45 mm, same other values | Factor 1.0, and 45 < 50, so the level stays **GREEN**. No SMS. |
| CULVERT: rain24 55 mm, rainHr 32, waterPct 70, reports 4 | rainHr 32 ≥ 30 and confirmed, so the level is **RED**. SMS, radio, volunteers and siren. If nobody acknowledges in 10 minutes, the District Committee and NDMA duty officer are called. |
| KROO: rain24 −5 mm (faulty gauge) | `isValidData` returns FALSE. No alerts this cycle. The error is logged for the duty officer, and previous levels stay in force. |

##  Sustainable Development Goals

- **SDG 11, Sustainable Cities and Communities** (Target 11.5): earlier, targeted warnings reduce deaths, injuries and losses from disasters.
- **SDG 13, Climate Action** (Targets 13.1 and 13.3): a repeatable early-warning logic that builds local capacity.
- **SDG 1, No Poverty** (Target 1.5): protects households with the least capacity to recover from flood shocks.
- **SDG 9, Industry, Innovation and Infrastructure** (Target 9.c): works on basic mobile phones with SMS, so no smartphone or data plan is required.

##  Open source and Digital Public Goods

- **Why open source:** a public-safety service should not depend on one vendor, and its rules must be open to inspection so that thresholds can be trusted and improved.
- **Benefits for Sierra Leone:** no licence fees, no vendor lock-in, skills for local developers and students, and reuse in other flood-affected districts.
- **Community contributions:** Krio, Mende and Temne translations, threshold calibration by hydrologists, ward and risk data, low-cost sensor integrations, testing and documentation.
- **Digital Public Good route:** target SDGs 11 and 13, use an OSI-approved licence (AGPL-3.0), publish anonymised alert logs as open data, use open standards such as REST/JSON and the Common Alerting Protocol, collect minimal personal data with consent, and prevent harm through validation, audit logs and escalation to humans. Formal DPG status requires nomination and review by the Digital Public Goods Alliance.
  ## SCREENSHOT
  <img width="1177" height="935" alt="image" src="https://github.com/user-attachments/assets/c6ed444c-fdf5-4bc8-99f3-f3c95f45b369" />
  <img width="1157" height="841" alt="image" src="https://github.com/user-attachments/assets/3b271023-b7ed-4401-bf13-b4df5bf07d06" />
  <img width="1655" height="966" alt="image" src="https://github.com/user-attachments/assets/b6340a67-a8ce-4d63-a2fa-dc30a6017edd" />





##  Limitations

- The thresholds are illustrative and must be calibrated with SLMet and NDMA.
- The Krio message templates must be written and approved by community translators before use.
## AUTHOR
EMMANUEL_ALVIN_MACARTHY_905006553





