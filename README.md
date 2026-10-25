# เว็บแอปต้นแบบ (Streamlit) — กลุ่ม 009-012 (ฉบับ V5)

## ไฟล์

| ไฟล์ | เนื้อหา |
|---|---|
| `app.py` | เว็บแอป รับตัวเลขดิบ 16 รายการ แล้วทำนายระดับความมั่นคงทางการเงิน |
| `models/S3_nn_model.pkcls` | Neural Network ของสถานการณ์ S3 จากวิดเจ็ต Save Model (NN) ใน `orange_S3.ows` (ฝึกด้วยชุดฝึก 25,940 รายการ) |
| `requirements.txt` | ไลบรารี Python ที่ล็อกเวอร์ชันแล้ว |
| `packages.txt` | แพ็กเกจระบบสำหรับ Streamlit Community Cloud |

## เวอร์ชันที่ใช้ (ล็อกใน requirements.txt)

| รายการ | เวอร์ชัน |
|---|---|
| Python | 3.12 (ทดสอบทั้งระบบ) · 3.13 ใช้ได้ (ทดสอบแบบจำลอง) |
| Orange3 | 3.40.0 |
| scikit-learn | 1.5.2 — ตรงกับเวอร์ชันที่บันทึกไว้ในไฟล์ `S3_nn_model.pkcls` |
| numpy / scipy / pandas | 2.3.5 / 1.18.1 / 2.3.3 |
| streamlit | 1.66.1 |
| PyQt5 / AnyQt | 5.15.11 / 0.2.2 |

ถ้าใช้ scikit-learn รุ่นอื่น ระบบจะเตือน `InconsistentVersionWarning` ตอนโหลดแบบจำลอง จึงล็อกให้ตรงกับรุ่นที่บันทึกแบบจำลอง
บน Streamlit Community Cloud ให้เลือก Python 3.12 ในหน้า Advanced settings ตอน Deploy

## การทำงาน

- ผู้ใช้กรอก 16 รายการ: ประชากรศาสตร์ 6 รายการ การเงินและสินเชื่อ 9 รายการ และคะแนนเครดิต โดยไม่ต้องกรอกอัตราส่วน
- เมื่อไม่มีสินเชื่อ ระบบกำหนด `loan_type = "None"` และตัวเลขสินเชื่อเป็น 0 ตรงกับข้อมูลฝึก
- แบบจำลองมีขั้นตอน One-hot และ Standardize จากวิดเจ็ต Preprocess อยู่ภายใน จึงส่งตัวเลขดิบเข้าได้โดยตรง
- แสดงระดับ ความน่าจะเป็นของแต่ละระดับ อัตราส่วนที่คำนวณจากตัวเลขดิบ และ "รหัสผลทำนาย" ภาษาอังกฤษ (ใช้ตรวจเทียบกับ Orange)

## การตรวจว่าให้ผลตรงกับ Orange (ไฟล์อยู่ใน `Orange Mining/results/`)

ทุกการตรวจรันในสภาพแวดล้อมที่ติดตั้งจาก `requirements.txt` นี้ (Python 3.12.3, Orange3 3.40.0, scikit-learn 1.5.2)

| ระดับการตรวจ | สคริปต์ | ผล |
|---|---|---|
| ไลบรารีของแอป | `app_env_check.py` | โหลดแบบจำลองได้โดยไม่มีคำเตือนเรื่องเวอร์ชัน และ scikit-learn ตรงกับที่บันทึกแบบจำลอง (`app_env_check.txt`) |
| แบบจำลองที่แอปโหลดใช้ | `app_model_check.py` | ป้อนชุดทดสอบ 6,484 รายการเข้าแบบจำลองโดยตรง ตรงกับ Orange 100% (`app_model_check.txt`) |
| ทั้งแอป (ช่องกรอก → แปลงค่า → ทำนาย → แสดงผล) | `app_pipeline_check.py` | ป้อนทีละรายการผ่านช่องกรอกของ `app.py` ด้วย `streamlit.testing` แล้วกดปุ่มทำนาย ตรงกับ Orange 6,484 / 6,484 รายการ (`app_pipeline_check.txt`, `.csv`) |
| หน้าเว็บในเบราว์เซอร์ | `app_fig3_local_check.js` | เปิดแอปด้วย Chromium กรอกตัวอย่างในรูปที่ 3 แล้วกดปุ่มทำนาย ได้ Moderate 87.25% ตรงกับ Orange (`app_fig3_local_check.png`, `.txt`) |

รันซ้ำ (จากโฟลเดอร์ `Orange Mining/results/`):

```
python app_env_check.py ../../App/models/S3_nn_model.pkcls ../../App/requirements.txt
python app_model_check.py ../../App/models/S3_nn_model.pkcls S3_test_predictions.csv
python app_pipeline_check.py ../../App/app.py S3_test_predictions.csv app_pipeline_check
# ตรวจผ่านเบราว์เซอร์: เปิด streamlit run ../../App/app.py --server.port 8599 ไว้ก่อน (ต้องมี Node.js และ playwright)
node app_fig3_local_check.js http://localhost:8599/ app_fig3_local_check.png app_fig3_local_check_page.txt
```

## เว็บที่เผยแพร่

https://ai-predicts-household-financial-stability0090012-2hyzabnonhyn3.streamlit.app/

ตัวอย่างสำหรับตรวจเว็บ = ตัวอย่างในรูปที่ 3 ของบทความ (แถวข้อมูลที่ 400 ของ `S3_test_predictions.csv` ค่าจริง Moderate)

| ช่องกรอก | ค่า | ช่องกรอก | ค่า |
|---|---|---|---|
| 1. อายุ | 29 | 9. เงินออมสะสม | 98716.51 |
| 2. เพศ | หญิง | 10. คะแนนเครดิต | 744 |
| 3. ระดับการศึกษา | ปริญญาโท | 11. มีการกู้สินเชื่อ | มี |
| 4. สถานะการทำงาน | ทำงานประจำ | 12. ประเภทสินเชื่อ | ยานพาหนะ |
| 5. อาชีพ | แพทย์ / บุคลากรทางการแพทย์ | 13. จำนวนเงินกู้ | 40517.86 |
| 6. ภูมิภาค | ยุโรป | 14. ระยะเวลากู้ (เดือน) | 180 |
| 7. รายได้ต่อเดือน | 1177.49 | 15. ค่างวดต่อเดือน | 607.56 |
| 8. รายจ่ายต่อเดือน | 815.60 | 16. อัตราดอกเบี้ย (%) | 16.44 |

ผลที่ต้องได้ (ตรงกับ Orange): ปานกลาง (Moderate) ความน่าจะเป็น Moderate 87.25% Stable 12.75% Unstable 0.00%

| การตรวจด้วยตัวอย่างนี้ | ผล | วันที่ตรวจ | หลักฐาน |
|---|---|---|---|
| แอปที่ติดตั้งจาก `requirements.txt` ที่ล็อกเวอร์ชัน (รันด้วย `streamlit run` แล้วเปิดในเบราว์เซอร์ Chromium) | ปานกลาง (Moderate) 87.25% · Stable 12.75% · Unstable 0.00% ตรงกับ Orange | 25 ต.ค. 2569 | `Orange Mining/results/app_fig3_local_check.png`, `.txt` |
| เว็บที่เผยแพร่ (หลังอัปเดต `requirements.txt` บน GitHub) | ______ | ______ | ภาพหน้าจอ ______ |

ขั้นตอนอัปเดตและตรวจเว็บที่เผยแพร่:

1. อัปโหลด `requirements.txt` ตัวนี้ทับไฟล์เดิมใน GitHub repository ของแอป (Commit changes) Streamlit จะติดตั้งไลบรารีใหม่และรีสตาร์ตแอปเอง URL เดิมไม่เปลี่ยน
2. รอจนหน้าเว็บโหลดเสร็จ (ครั้งแรกอาจใช้หลายนาที) ถ้าติดตั้งไม่ผ่าน ให้ลบแอปแล้ว Deploy ใหม่โดยเลือก Python 3.12 ใน Advanced settings
3. เปิดเว็บ กรอกค่าตามตารางด้านบน กดปุ่มทำนาย แล้วบันทึกภาพหน้าจอ
4. กรอกผลและวันที่ในแถว "เว็บที่เผยแพร่" ของตารางนี้

> ใช้เพื่อการศึกษาเท่านั้น แบบจำลองสอนจากข้อมูลสังเคราะห์ ไม่ใช่คำแนะนำทางการเงิน
