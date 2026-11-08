# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** อภิวัชร ผดุงอรรถ

**Student ID / รหัสนิสิต:** 6970314621

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large |511 |616 |93.50% |100% |
| t0-small |31 |60 |72.50% | 0%|

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

>Large Model ให้ Accuracy เพิ่ม แต่ต้องแลกด้วย Cost และ Latency ที่เพิ่มมากขึ้นด้วย

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before |26.5| 236| 374| 18| 0| 93.40%|
| t1-after | 10.1| 414| 1019| 0| 22| 92.70%|

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

>Column run time และ errors บอกความจริง t1-before เกิด error และ run time มาก ซึ่งตัว latency (p50/p95) จะถูกนับเฉพาะตัวที่ pass เท่านั้น

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

>การเชื่อมต่อไปยัง large model ถูก call ที่ค้างถือรอไว้เฉยๆ ทำให้ส่งผลกระทบต่อ request อื่น

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

>จัดเป็น queue ไว้ตาม topology 4 Asynchronous / batch โดย review pending จะถูก batch job ของระบบนำไปติด label

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

>- 0.1 วินาทีทำให้ process จบไวเกินไป (pending ทั้งหมด 200/200 request) และการใช้ large call ทำให้เปลือง cost
>- timeout 10 วินาที ไม่ส่งผลอะไรเนื่องจาก timeout ของระบบเป็น 10 วินาทีอยู่แล้ว อ้างอิงจากผลของ t1-before ที่ให้ค่า accuracy ที่เท่ากัน
  

## Task 2: Cascade (copy the Task 2 results from results/runs.csv here / copy ผลลัพธ์ของ Task 2 จาก results/runs.csv มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 |17 |539 |91.50% | 38%|
| t2-0.7 | 0.7 |338 |574 |93.00% | 53%|
| t2-0.8 | 0.8 |443 |604 |93.50% | 64%|
| t2-0.9 | 0.9 |490 |613 |93.50% |77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

>threshold 0.6 เนื่องจากมีการใช้ large call ที่น้อยมาก ได้ latency ต่ำที่สุด อาจจะต้องยอมเสีย accuracy บางส่วน แต่ผลลัพธ์ที่ได้ก็มากพอๆกับ threshold ที่ใช้ large call เยอะๆ

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

>

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

>

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | | | | | |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

>

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

>

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

>
