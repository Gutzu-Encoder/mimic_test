# mimic_test
ในโปรเจคนี้เราใช้ data จาก mimic iii โดยเลือกตัว discharge summary มาคลีนเพื่อเอา leakage ออกด้วย regex เเล้วประเมินด้วย llm judge 2 ตัวคือ dsk v4 pro think high กับ glm 5.1 ประเมินโมเดล medgemma 1.5 4b/ qwen 3.6 -35b 4bit / llama-dsk distilalte 8b 
สาเหตุที่เลือก
- medgemma 1.5 4b: ตาม hf กล่าวว่ามี performanve ดีกว่า medgemma 1  24b เเละ compute ถูกกว่า สามารถติดตั้งใน T4 ได้ นำมาเป็น baseline model
- deepseek distil llama 8b: มีการทำ reasonnig ในขนาดย่อมเยา
- qwen 3.6 35B - 5bit: สามารถ fit ใน A100 ได้ เเละมีการ reasoning mี่ค่อนข้างดี

ก่อนหน้านี้มีการใช้ qwq 32b เเต่เนื่องจากตอนนั้น data clean พลาดเเละ inference พลาด ณ จุดที่พิมพ์ compute collab pro ได้สิ้นลงเเล้ว
ส่วนตัว api ไม่ได้เอามาเทสเพราะมีกังวลว่าจะมีการซ้ำซึ่งจะกล่าวต่อไป เเละไม่ได้ใช้ฝั่งอเมริกาเพราะราคาสูงเเละนี่เป็นช่วงการทดลองเท่านั้น

 judge คือ glm 5.1 think เเละ deepseek v4 pro thinking

สิ่งที่ทำคือการพยายามปรัเมิน 2 ส่วนเเยกกัน
- reasoning โดยเจาะจงคือ clinical reasoning สามารถตรวจสอบได้จาก judge prompt 
- accuracy

โดยในส่วน reasoning task เราจะมีการใช้ตัว 
- negated คือการใช้ regex มาเติม no หน้า keyword สำคัญเช่น มีโรคมั้ย ถ้า text บอกมี เติมเข้าไปจะไปเป็นไม่มี
- counterfactual คือการ inject ข้อความที่เเสดงให้ผิดอย่างเห็นๆ
- random masked คือจะมีการใช้ regex สลับคำที่สำคัญทางการเเพทย์เเบบสุ่มด้วย random เเทนด้วย masked

เเละใน accuracy จะมีการทำ
- 0 shot
- 1 shot

โดยระหว่างการทดลองได้พบอะไรน่าสนใจหรือติดปัญหาคือ
- ในการดำเนินการมีการลองผิดลองถูกจำนวนมาก compute ที่ใช้คือ ทั้ง 100 compute ของ collab pro/ glm 5.1 api/ deepseek api/ kimi k2.6 api ใช้เวลาทำ 2 เดือน มีการลองโมเดลหลายตัวเเละวิธีการประเมินต่างๆ ตอนเเรกได้ใช้ deepseek v4 pro think ด้วยเเต่ติดปัญหาที่ว่า จากเปเปอ [Wataoka et al. (2024/2025)] (arXiv:2410.21819): Self-Preference Bias in LLM-as-a-Judge
llm ที่ประเมินเครือตัวเองอาจจะเอื้อผลที่ดีขึ้น
เช่นผลจาก deepseek v4 นั้นใน task counterfact/ negate/ mask สามารถ perform ออกมาได้ accuracy  ตรงหมดทั้งที่เนื้อความเปลี่ยนไปเเล้ว สิ่งนี้ทำให้มีการปรับการทดลองไม่วัด accuracy ตัว 3 แนืกระรนืห ouh
  


- มีอีกเรื่องที่ต้องปรับมาใช้ 2 judge นั้นคือต่อเน่องมาจากการที่ self-preference พบว่า model deepseek-distil-llama 8b เเละ mode medgemma 1.5 4b มีคำตอบเดียวกันคือ metastatic beast cancerซึ่งเเม้ไม่ตรงเเต่เกี่ยวข้อง เเต่ judge deepseek ให้ llama dsk 2
ในขณะที่ medgemma ให้ 0 ทำให้อาจจะต้องเเยกปัญหา 2 ส่วน
   - input text: ตอนเเรกใส่ทั้งคำตอบไปเเต่ ถ้าจะวัดเเค่ accuracy ควรเป็น final outcome
       - เลยปรับให้มีการ extract ตัว principal diagnosis อีกทีด้วย regex เเละ deepseek v4 flash
       - เพิ่ม judge อีก 1 คือ glm 5.1 สาเหตุเพราะถูก เเม้ kimi k2.6 จะกล่าวขานว่า performance ดีกว่าเเต่ในการคิดมันนั้น context length ไม่มากเเละมันใช้ token เยอะในการคิด ถ้าจะประเมินทุก text จากหลายๆโมเดลทั้ง accuracy reasoning ซึ่งรับทั้ง discharge summary ไปด้วยคงจะเป็นการลำบาก เเม้จะมี prompt cache ก็ตาม

- เเต่มีผลที่น่าสนใจที่ไม่ได้ใส่มาคือใน counterfactual จะพบว่าตัว deepsee v4 pro think มีการ flag ว่า " text { ตัว counterfact}. มีการใส่เข้ามาโดยไม่ีเกี่ยวกับ context เลยเเละคิดว่าน่าจะเป็น noise เป็นที่น่าตั้งคำถามว่าจะ model นั้นมีความทนทานต่อ noise เเค่ไหน ตัวนี้อาจจะไม่ใช่ข้อดีเพราะถ้าตั้งข้อสันนิษฐานว่า " ถ้า model's weight รู้จัก mimic อยู่เเล้ว?" เเน่นอนว่าการเทสสิ่งนี้ก็ควรเทสด้วย unique OOD เเต่ไม่มี

- เเละอีกอย่างคือ medgemma ในตัว counterfactual มีการเข้า self correction คือโมเดลพิมพ์ออกมาตาม system prompt คือเป็น 1. 2. 3. เเล้วลงคำตอบเเต่ก็จะเริ่มมีการเข้าสู่ self correction loop คือ คิดว่ามันใช้รึเปล่า เเล้ววนเร่อยๆไม่มีคำตอบสุดท้ายเลยคิดว่าเป็นไปได้มั้ยว่าจะไม่ได้เอื้อ model's weight ขนาดนั้น 

- เเละเมื่อพูดถึง prompt template กระบวนการคิด dee[seek 4 pro เเละ deepseek disitlalte llama 8b จะพบว่ามันคิดตามของมันก่อน เเล้วค่อยเติม template ทีหลัง เหมือนเด็กที่ทำการบ้านคณิตว่า จงเเสดงวิธีทำ ยังงี้ step 1 2 3 มีการคิดของมันเองเเล้วค่อยเติมเองให้สอดคล้อง 

โดย workflow คือ
นำ data มา 5 topics เเละเลือก topis ละ 2 hdmd โดย 1 hdmd จะรัน 0 shot/ 1shot/ counterfactual/ negated/ random masked [5 conditions] รวมเเล้ว 50 inference call / model
จะคลีนดาต้าด้วยการ
- ใช้ regex เอาคำที่ไม่สำคัญออก
- ใช้ regex ตัดตัว \n ออก
- เพื่อคุม token เราจะใช้การใช้ data structure queue โดย 1. ไล่ตาม topic => 2. set while จนกว่าจะเต็ม => 3. นำ cleaned ที่ token ถึงตาม threshold มาเข้า เมื่อเต็ม topic นึง => move ไปจนกว่าจะครบ
- นำ long title มาเป็น ground truth

ส่วนการ augmented ก็ตามที่ได้กล่าวไป
เเละต่อไปจะมีการ inference โดยจะนำ ผลที่ได้มา เข้า regex/ deepseek v4 flash เพื่อหา principal diag มาคำนวณตัว accuracy
เเต่การ inference มีปัญหาคือว่า model final result มีการ duplicate คือต่อ hdmhd มันคิดซ้ำเพราะยังงั้นเลยมีการ inferenve ใหม่อีกรอบโดยการเขียนให้มันไม่เอาตัวที่มีอยู่เเล้วเเละรันที่เหลออีกครึ่งนึง


