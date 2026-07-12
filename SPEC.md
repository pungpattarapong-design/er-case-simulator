# ER Case Simulator — Build Instructions

## What to Build
Single `index.html` file with all CSS/JS inline. ER attending physician case simulator for medical students.

**Output**: `/tmp/ai-orchestrator-workspace/index.html`

## Architecture
- Single file: `index.html` with inline `<style>` and `<script>`
- No frameworks, no CDN, no external dependencies
- Dark theme: bg `#0f1923`, cards `#162231`, accent `#3498db`, warn `#e67e22`, danger `#e74c3c`, success `#27ae60`
- Max-width 1080px, responsive (stack on mobile <768px)
- localStorage for progress persistence

## UI Layout
1. **Header**: "ER Case Simulator" + subtitle
2. **Case Selection**: Grid of 6 case cards
3. **Chat View**: Messages with "Dr. Staff" (left, `#1a2a3a`) and "You" (right, `#1d3b5c`)
4. **Input**: Text input + Send button, Enter to send
5. **Score Screen**: After case completion

## Case Engine
- Each case = array of phases
- Each phase: attending says something → student responds → keyword match → feedback + next phase
- Keyword matching: case-insensitive, normalize input, check if ANY keyword present, pick highest-scoring match
- Fallback: if no keywords match, attending asks a clarifying question

## 6 Cases

### Case 1: "Chest Pain — 55M" (difficulty: medium)
A 55M smoker with HTN+DM presents with substernal chest pressure for 2 hours, radiating to left arm, diaphoretic. ECG shows ST elevation II,III,aVF → Inferior STEMI.

**Phase 1 (Chief Complaint)**: "A 55-year-old man presents with chest pain for 2 hours. What's your first question?"
- Keywords: ["character","describe","quality","what kind"] → "Good. He describes it as pressure, like an elephant sitting on my chest — 8/10, substernal, radiating to left arm." → next
- Keywords: ["onset","when","start"] → "Relevant. It started 2 hours ago while watching TV." → next
- Keywords: ["risk","past","cardiac","htn","dm","smoking"] → "Important — but first characterize the pain." → same phase
- Fallback: "Start with the character of the pain — what does it feel like?"

**Phase 2 (Risk Factors)**: "He's 55, smoker, known HTN and DM. What else do you want to know?"
- Keywords: ["associated","symptom","nausea","sweat","diaphoresis","sob","dyspnea"] → "He's diaphoretic, mildly nauseated, no vomiting. No dyspnea at rest." → next
- Keywords: ["vital","bp","hr","rr","sats"] → "Vitals: BP 148/92, HR 102, RR 20, SpO2 96% on room air, temp 36.8" → next
- Fallback: "Think about associated symptoms — nausea, diaphoresis, dyspnea?"

**Phase 3 (ECG)**: "You order an ECG. It shows ST elevation in leads II, III, aVF — 3mm. What's the diagnosis?"
- Keywords: ["inferior","stemi","mi","myocardial","infarction","heart attack"] → "Correct — Inferior STEMI. Now, what's your immediate management?" → next
- Keywords: ["right","rv","v4r","posterior","v7","v8","v9"] → "Good thinking — you want a right-sided ECG to check for RV involvement. Let's say V4R shows 1mm elevation. Now what's your management?" → next
- Fallback: "Look at the leads — II, III, aVF. What territory is that?"

**Phase 4 (Management)**: "What medications do you give now?"
- Keywords: ["aspirin","asa","antiplatelet","300","325"] → "Aspirin 300mg PO — good. What else?" → this phase stays but moves forward
- Keywords: ["ticagrelor","clopidogrel","p2y12","antiplatelet","dual"] → "Right. Ticagrelor 180mg or Clopidogrel 600mg loading." → this phase stays
- Keywords: ["heparin","anticoagulation","enoxaparin","lmwh","ufh"] → "Yes — anticoagulation. Enoxaparin or UFH." → this phase stays
- Keywords: ["morphine","pain","analgesia"] → "Morphine for pain — but use cautiously, it can cause hypotension." → this phase stays
- Keywords: ["oxygen","o2","supplemental"] → "Only if SpO2 < 90%. He's at 96% — no O2 needed." → this phase stays
- Keywords: ["nitroglycerin","ntg","gtn","nitrate"] → "NTG — but careful with inferior MI, risk of hypotension from RV involvement." → this phase stays
- When student has covered 3+ meds → "Good. MONA-BASH: Morphine, Oxygen (if hypoxic), Nitrates, Aspirin + Beta-blocker, ACE-I, Statin, Heparin. Now — reperfusion strategy?" → next

**Phase 5 (Reperfusion)**: "How do you reperfuse?"
- Keywords: ["pci","cath","angioplasty","stent","catheterization","primary"] → "Primary PCI if door-to-balloon <90 min. We're at a community hospital without cath lab — what now?" → next
- Keywords: ["thrombolytic","fibrinolytic","streptokinase","tpa","tnk","tenecteplase"] → "Yes. Tenecteplase weight-based bolus. But first — any contraindications?" → next
- Fallback: "Primary PCI is gold standard. But if no cath lab within 90 minutes?"

**Phase 6 (Disposition)**: "Patient received TNK, pain improving, ST resolution 50%. Where does he go?"
- Keywords: ["ccu","icu","cardiac","monitor","admit"] → "CCU admission — correct. He needs continuous monitoring, serial ECGs, and transfer to PCI-capable center if needed. Good case!" → end
- Fallback: "He needs monitored bed — which unit?"

**End Message**: "Well done! Key points: Inferior STEMI recognition, right-sided ECG, MONA-BASH, and reperfusion strategy. Always check for RV involvement in inferior MI."

---

### Case 2: "Anaphylaxis — 35F" (difficulty: medium)
35F develops urticaria, stridor, wheezing, hypotension 85/50 15 min after amoxicillin.

**Phase 1 (Recognition)**: "35F just received amoxicillin for UTI. She develops diffuse urticaria, stridor, wheezing. BP 85/50, HR 120. What's happening?"
- Keywords: ["anaphylaxis","anaphylactic","allergic","shock","reaction"] → "Anaphylaxis — correct. What's your FIRST action right now?" → next
- Fallback: "Look at the timeline: drug → minutes later → skin + respiratory + hypotension. What syndrome is this?"

**Phase 2 (First Action)**: "She's deteriorating. What do you do RIGHT NOW?"
- Keywords: ["adrenaline","epinephrine","epi","im","intramuscular","0.5"] → "Yes! Adrenaline 0.5 mg IM (0.5 mL of 1:1000) anterolateral thigh. Adult dose. Where do you inject?" → next
- Keywords: ["airway","intubate","oxygen","o2"] → "She has stridor — airway concern is valid. But before intubation, what's the life-saving drug?" → same phase
- Keywords: ["fluid","iv","nss","bolus"] → "Fluids are important — but the priority drug is adrenaline. What dose?" → same phase
- Fallback: "The priority drug for anaphylaxis — injected into muscle. What is it?"

**Phase 3 (Adrenaline Details)**: "Anterolateral thigh. When can you repeat the dose?"
- Keywords: ["5","15","min","repeat","every","q5","q15","5-15"] → "Correct — every 5-15 minutes if no improvement, up to 2-3 doses. Now — adjunct medications?" → next
- Fallback: "Every 5-15 minutes, up to 3 doses if no response."

**Phase 4 (Adjuncts)**: "What else do you give?"
- Keywords: ["antihistamine","cpm","chlorpheniramine","diphenhydramine","h1"] → "CPM 10mg IV or diphenhydramine — good. What about steroids?" → this phase stays
- Keywords: ["steroid","dexamethasone","hydrocortisone","methylprednisolone"] → "Dexamethasone or hydrocortisone IV. Won't help acutely but prevents biphasic reaction." → this phase stays
- Keywords: ["salbutamol","neb","bronchodilator","beta","agonist"] → "Salbutamol nebulized for bronchospasm — yes. What about fluids?" → this phase stays
- Keywords: ["fluid","bolus","nss","crystalloid","1-2","liter","20","ml/kg"] → "NSS 1-2L bolus. She's hypotensive — fluid resuscitation is key." → this phase stays
- When 2+ covered → "Good. Summary: Adrenaline is #1, then antihistamine, steroid, bronchodilator, and fluids. Now — where does she go?" → next

**Phase 5 (Disposition)**: "She improves after 2 doses of adrenaline. Where to?"
- Keywords: ["observe","observation","4-6","6-8","hour","biphasic","admit","ward"] → "Admit for observation — risk of biphasic reaction within 4-8 hours. Never discharge immediately after anaphylaxis. Well done!" → end
- Fallback: "Risk of biphasic reaction — she needs observation for at least 4-6 hours."

---

### Case 3: "Sepsis — 68M" (difficulty: medium)
68M, fever 3 days, confused, BP 88/54, HR 110, RR 28, lactate 4.2. Suspected UTI.

**Phase 1 (Screening)**: "68M with fever, confusion, hypotension. What screening tool do you use?"
- Keywords: ["sirs","qsofa","news","sepsis","screen"] → "He meets SIRS (HR>90, RR>20), qSOFA 2 (confusion, RR≥22, SBP≤100). This is sepsis screening positive. What's the diagnosis?" → next
- Fallback: "Use SIRS criteria or qSOFA to screen for sepsis."

**Phase 2 (Severity)**: "BP 88/54 after 30mL/kg fluids, lactate 4.2. Diagnosis?"
- Keywords: ["septic","shock","sepsis","refractory","vasopressor"] → "Septic shock — persistent hypotension after fluids + lactate >2. What's the first hour bundle?" → next
- Fallback: "Hypotension not responding to fluids + elevated lactate = what?"

**Phase 3 (Hour-1 Bundle)**: "What's in the Hour-1 bundle?"
- Keywords: ["lactate","blood culture","antibiotic","fluid","crystalloid","30","ml/kg"] → "Correct: measure lactate, blood cultures x2, broad-spectrum antibiotics, 30mL/kg crystalloid, vasopressors if needed. All within 1 hour. What antibiotics?" → next
- Fallback: "Lactate, blood cultures, antibiotics, fluids, vasopressors — all in the first hour."

**Phase 4 (Vasopressor)**: "She remains hypotensive after fluids. Which vasopressor?"
- Keywords: ["norepinephrine","noradrenaline","levophed","ne"] → "Norepinephrine 4mg in 5%DW 250mL, start 5-15 mL/hr, titrate to MAP ≥65. If NE dose exceeds 0.25 mcg/kg/min — what do you add?" → next
- Keywords: ["dopamine","vasopressin","adrenaline"] → "Norepinephrine is first-line. Dopamine is second-line. NE is preferred." → same phase
- Fallback: "First-line vasopressor for septic shock — starts with 'N'."

**Phase 5 (Refractory Shock)**: "NE at 0.3 mcg/kg/min, still hypotensive. What now?"
- Keywords: ["hydrocortisone","steroid","corticosteroid","50","100","200"] → "Hydrocortisone 100mg IV stat, then 200mg/24hr. Check serum cortisol first. SEPSIS-3 recommends steroids for refractory shock. Where does this patient go?" → next
- Fallback: "Add stress-dose steroids — hydrocortisone."

**Phase 6 (Disposition)**: "Where do you admit?"
- Keywords: ["icu","intensive","critical"] → "ICU — correct. Septic shock with high vasopressor requirement needs intensive monitoring. Good case!" → end

---

### Case 4: "Acute Asthma — 22F" (difficulty: easy)
22F, acute dyspnea, wheezing, can only speak in words, PEFR 35% predicted, SpO2 89%.

**Phase 1 (Severity)**: "22F with acute asthma. She can only say single words, using accessory muscles, SpO2 89%. How severe?"
- Keywords: ["severe","life threatening","acute severe","status"] → "Acute severe asthma — possibly life-threatening with that SpO2. What's your first treatment?" → next
- keywords: ["life-threatening"] → "Correct — SpO2 <92% makes this life-threatening. What's your first treatment?" → next
- keywords: ["moderate","mild"] → "Look again — single words, accessory muscles, SpO2 89%. This is severe." → same phase
- Fallback: "She can't complete sentences, SpO2 <92% — this is acute severe, possibly life-threatening."

**Phase 2 (Treatment)**: "What do you do immediately?"
- Keywords: ["oxygen","o2","supplemental","high flow","target"] → "Oxygen to target SpO2 94-98% — correct. What nebulized medications?" → this phase stays
- Keywords: ["salbutamol","albuterol","saba","beta","agonist","neb","nebulized"] → "Salbutamol 5mg nebulized, continuous or back-to-back. What do you add?" → this phase stays
- Keywords: ["ipratropium","atrovent","anticholinergic"] → "Ipratropium 0.5mg nebulized — yes, combined with SABA in severe exacerbations." → this phase stays
- Keywords: ["steroid","prednisolone","hydrocortisone","methylprednisolone","iv","oral"] → "Steroids — oral prednisolone 50mg or IV hydrocortisone 200mg. They take hours but reduce relapse." → this phase stays
- Keywords: ["magnesium","mgso4","mg"] → "IV magnesium 2g over 20 minutes — good for severe cases. Now reassess." → this phase stays
- When 2+ covered → "Good. Key treatment: O2, SABA+ipratropium nebs, systemic steroids, consider IV Mg. Reassess in 1 hour." → next

**Phase 3 (Reassessment)**: "After treatment, PEFR now 55%, still wheezing but speaking sentences. Next step?"
- Keywords: ["admit","ward","observation","discharge"] → "Admit for ongoing nebs and monitoring. Consider ward admission. Discharge criteria: PEFR >75%, off O2, stable 1hr post-treatment. Good case!" → end
- Fallback: "She improved but still <75% predicted. She needs admission for ongoing treatment."

---

### Case 5: "Pelvic Pain – 29F" (difficulty: easy)
29F, suprapubic pain + spotting. Cycle day 18, afebrile, UPT negative, UA normal.

**Phase 1 (Chief Complaint)**: "29F with lower abdominal pain and spotting for 1 day. What's your FIRST question?"
- Keywords: ["lmp","menstrual","period","cycle","last","date"] → "LMP was 3-8 May. Today is cycle day 18. Why is this important?" → next
- Keywords: ["pregnant","pregnancy","sexual","contraception"] → "Good — you must rule out pregnancy. UPT is negative. She denies pregnancy risk. Now — about her cycle?" → this phase stays
- Fallback: "In a reproductive-age woman with pelvic pain and bleeding — what's the most important history?"

**Phase 2 (Cycle Day)**: "Cycle day 18 — mid-cycle. What does that suggest?"
- Keywords: ["ovulation","mittelschmerz","mid cycle","follicular","rupture","corpus","luteum"] → "Mittelschmerz — ovulation pain. Mid-cycle follicular rupture causes peritoneal irritation and spotting from the estrogen dip. Classic presentation." → next
- Keywords: ["ectopic","pid","cyst"] → "Those are important differentials — but what about the timing? Day 18 = mid-cycle. Think about ovulation." → same phase
- Fallback: "Day 14-18 is mid-cycle — what physiological event happens then?"

**Phase 3 (Differential)**: "What's your differential?"
- Keywords: ["mittelschmerz","ectopic","pid","cyst","uti","corpus","luteum"] → "Good differential. Mittelschmerz is most likely given the mid-cycle timing + suprapubic pain + spotting. What investigations help rule out the dangerous ones?" → next
- Fallback: "Consider: Mittelschmerz, ectopic pregnancy, PID, corpus luteum cyst, UTI."

**Phase 4 (Investigations)**: "What tests do you order?"
- Keywords: ["upt","urine","pregnancy","beta","hcg"] → "UPT negative — reassuring but not 100% for ectopic. If clinical suspicion, get quantitative β-hCG." → this phase stays
- Keywords: ["ua","urinalysis","uti","urine"] → "UA normal — no pyuria, no hematuria. UTI ruled out." → this phase stays
- Keywords: ["ultrasound","us","pelvic","transvaginal","tvs"] → "Pelvic US — no free fluid, no adnexal mass. This rules out ectopic and hemorrhagic cyst." → this phase stays
- Keywords: ["pelvic","exam","speculum","bimanual","cervical","motion","cmt"] → "Pelvic exam — important for PID. If no CMT, no discharge, PID is unlikely." → this phase stays
- When 2+ covered → "With negative UPT, normal UA, no free fluid on US — ectopic and cyst are unlikely. Mittelschmerz is the diagnosis." → next

**Phase 5 (Management)**: "What do you tell the patient?"
- Keywords: ["reassure","benign","normal","self","limit","analgesia","nsaid","paracetamol","pain","relief","observation"] → "Reassure — this is benign, self-limited. Paracetamol or NSAIDs. Return if worsening pain, heavy bleeding, fever, dizziness." → next
- Fallback: "Mittelschmerz is benign and self-limited. What advice do you give?"

**Phase 6 (Teaching)**: "Final teaching point — what's the pathophysiology?"
- Keywords: ["follicle","rupture","peritoneal","irritation","estrogen","dip","spotting","prostaglandin"] → "Exactly. Follicular rupture causes peritoneal irritation from follicular fluid/blood. The pre-ovulatory estrogen dip causes endometrial spotting. Classic timing is key. Great case!" → end
- Fallback: "Follicular rupture irritates the peritoneum. The estrogen dip causes spotting. Timing (day 14-18) is diagnostic."

---

### Case 6: "Trauma — 25M" (difficulty: hard)
25M motorcycle vs car, helmeted. GCS 9 (E2,V2,M5), abrasions left chest, decreased breath sounds left, trachea deviated RIGHT. BP 80/40, HR 130.

**Phase 1 (Primary Survey)**: "25M motorcycle crash. He's groaning, opens eyes to pain, localizing. GCS is 9. What's your first step?"
- Keywords: ["abcde","primary","survey","airway","c-spine","cervical","spine"] → "ATLS primary survey — ABCDE with C-spine protection. Start with A: what's the airway status?" → next
- Fallback: "ATLS approach — what comes first?"

**Phase 2 (Airway)**: "He's groaning — airway is patent but unprotected at GCS 9. What do you do?"
- Keywords: ["intubate","intubation","ett","rsi","definitive","airway","tube"] → "GCS ≤8 is the classic intubation threshold. At GCS 9, you're right to be concerned. Let's move to B — what do you find?" → next
- Keywords: ["observe","gcs","protect","monitor","suction"] → "At GCS 9 with decreasing trend, you should prepare for intubation. But let's move to Breathing." → next
- Fallback: "GCS 9 — is the airway protected? What's the intubation threshold?"

**Phase 3 (Breathing — Tension Pneumothorax)**: "Decreased breath sounds left, trachea deviated RIGHT, distended neck veins, hypotensive. What's the diagnosis?"
- Keywords: ["tension","pneumothorax","tension pneumo","ptx"] → "Tension pneumothorax — clinical diagnosis. What do you do RIGHT NOW?" → next
- Fallback: "Tracheal deviation AWAY from decreased breath sounds + hypotension + distended neck veins = what?"

**Phase 4 (Immediate Action)**: "What's the immediate management?"
- Keywords: ["needle","decompression","needle thoracostomy","14g","16g","2nd","intercostal","midclavicular","5th","anterior","axillary"] → "Needle decompression: 14-16G angiocath, 2nd intercostal space midclavicular line (or 5th ICS anterior axillary line). You hear a rush of air. BP improves to 100/60. What now?" → next
- Keywords: ["chest","tube","thoracostomy","icd","drain"] → "Chest tube is the definitive treatment — but first, what's the immediate life-saving intervention? Needle decompression buys time." → same phase
- Fallback: "He's dying. You need immediate decompression. How?"

**Phase 5 (Definitive Care)**: "Needle decompression done. BP now 100/60, SpO2 92%. What now?"
- Keywords: ["chest tube","thoracostomy","icd","tube","thoracostomy","28","32","fr"] → "Chest tube — 28-32Fr in 5th ICS anterior axillary line. Connect to underwater seal. After chest tube — what's next in the survey?" → next
- Fallback: "Needle decompression is temporizing. What's the definitive intervention?"

**Phase 6 (Continue Survey)**: "Chest tube in place. What now?"
- Keywords: ["circulation","c","iv","access","fluid","resuscitation","bleeding","shock","hemorrhage","transfusion"] → "Circulation — 2 large-bore IVs, start fluids. He's in hemorrhagic shock (Class III — HR>120, BP decreased). What about D and E?" → next
- Keywords: ["secondary","survey","head","toe","expose"] → "Yes — after primary survey and resuscitation, move to secondary survey. But first check D and E." → same phase
- Fallback: "Primary survey continues — what's after A and B?"

**Phase 7 (Disposition)**: "CT shows small pneumothorax with chest tube in good position, no other injuries. Where to?"
- Keywords: ["icu","trauma","surgery","admit","monitor","observe"] → "ICU or trauma surgery admission. He needs continuous monitoring, pain control, and serial exams. Excellent trauma case!" → end

## Keyword Matching Implementation
```javascript
function matchResponse(input, expectedResponses, fallback) {
  const normalized = input.toLowerCase().trim();
  let bestMatch = null;
  for (const resp of expectedResponses) {
    for (const kw of resp.keywords) {
      if (normalized.includes(kw.toLowerCase())) {
        if (!bestMatch || resp.score > bestMatch.score) {
          bestMatch = resp;
        }
      }
    }
  }
  return bestMatch || { feedback: fallback, score: 0, next_phase: null, keywords: [] };
}
```

## Scoring
- Track cumulative score in localStorage
- Show at end: score/max + qualitative feedback
- Messages array also stored in localStorage

## Edge Cases
- Empty input → ignore
- Long input → display full, truncate at 500 chars in chat bubble
- Refresh → restore from localStorage
- localStorage unavailable → graceful degradation (no persistence)
- All cases completed → congratulations message

## Important
- System messages (phase transitions) = centered, italic, muted
- Typing indicator before attending messages (500ms delay for realism)
- Mobile: input area fixed at bottom
- Desktop: chat area scrollable with sticky input
