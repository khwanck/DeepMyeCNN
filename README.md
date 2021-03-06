## Group DeepMye

# Topic: Thai Fruits Classification

## Highlight:
*	สามารถพัฒนาโมเดลให้สามารถทำนายผลไม้ไทย 8 ชนิดจากรูปได้
*	ผลการทำนายเป็นที่น่าพอใจโดยมีความแม่นยำมากกว่า 90% (จาก test data set)
*	มีฟังก์ชันช่วยยืนยันว่าการทำนายนำข้อมูลมาจากส่วนไหนของรูปภาพ (GradCAM)

## Introduction:
  ทางกลุ่มของเราทำ single-label multi-classification โดยหลังจากที่ทางกลุ่มได้พิจารณา image class ใน image net dataset แล้วพบว่า มี class เกี่ยวกับ ผลไม้ประมาณ 8 classes และ แต่ละ class นั้น เป็นผลไม้จากต่างประเทศ หรือผลไม้ที่เป็นที่รู้จักกันทั่วๆไป เช่นแอปเปิ้ล, สตรอเบอรี่, กล้วย, ส้ม, เลม่อน ฯ
แต่ไม่มีผลไม้ไทยเลย และจากการที่ได้ลองนำภาพของผลไม้ไทยใส่ไปเผื่อให้ทำนายออกมานั้น พบว่าความแม่นยำค่อนข้างต่ำ ทางกลุ่มจึงอยากจะทำ classification model ขึ้นมาเพื่อลองใช้กับผลไม้ของไทยๆบ้าง

## Data Source:
  ข้อมูลท่างกลุ่มใช้ คือไม้ไทยชนิดต่างๆ ทั้งสิ้น 8 ชนิด ได้แก่ ทุเรียน, น้อยหน่า, ลิ้นจี่, มังคุด, มะละกอ, ชมพู่, มะเฟือง และ มะม่วง โดยทางกลุ่มได้ไปรวบรวมภาพมาจากทาง Internet จำนวนชนิดละ 200 ภาพ
ภาพที่ได้มาจะมีขนาดของภาพที่แตกต่างกัน ทางกลุ่มได้ทำการ reshape เป็นขนาด 256X256 และภาพบางภาพที่ได้มาอาจจะมี format ของภาพที่ไม่สามารถนำไปใช้งานได้ จึงได้ทำการเปลี่ยนให้เป็นformat เดียวกันทั้งหมดเพื่อให้ง่ายต่อการใช้งาน
ขั้นตอนที่จะต้องเพิ่มขึ้นมาจากงาน classification ทั่วไปคือเราต้องเขียนฟังชั่นเพื่อทำการ label class ของรูปแต่ละรูปด้วยเพื่อนำไปเป็นผลเฉลยสำหรับการ train และเปรียบเทียบผลจากการทำนาย

## Network architecture:
  เราได้ทดลองนำโมเดลต่างๆมาเป็น Pre-training model เช่น VGG16, MobileNetV2, DenseNet201 เป็นต้น สุดท้ายเราพบว่าโมเดล DenseNet201 ให้ผลการทำนายที่ดีที่สุดเราจึงเลือก DenseNet201 มาเป็น Pretraining 
โดยเราจะนำ output ที่ได้จาก DenseNet201 มาทำ conv2d และ maxpooling2d อีกรอบนึงแล้วค่อยทำ Flatten และ dense กับ dropout อีกอย่างละ 2 layers ก่อนจะ dense และ Softmax เป็นครั้งสุดท้ายเพื่อเป็นผลการทำนายของ class ทั้ง 8 classes
การที่เรานำมา conv2d ต่ออีกครั้งเพื่อจะเป็นการง่ายต่อการเทรนและสามารถนำไปใช้สำหรับ GradCAM ได้ง่ายอีกด้วย

![image](https://github.com/khwanck/DeepMyeCNN/blob/main/images/DenseNet201.png)

## Training:
  เราได้ทำการแบ่งข้อมูลจาก data set มา 90% หรือประมาณ 1440 รูป เพื่อนำมาเป็นข้อมูลที่จะใช้เทรนโมเดลเรา โดยจะมี hyper parameter ต่างๆดังนี้  optimizers = Adam , Learning rate = 0.0001,batch_size = 20, epochs = 10

![image](https://github.com/khwanck/DeepMyeCNN/blob/main/images/training-parameter.png)

เนื่องจากผลการเทรนได้ผลลัพธ์เป็นที่น่าพอใจเราจึงไม่จำเป็นต้องปรับ hyper parameter อื่นๆอีก

## Results:
  หลังจากได้modelที่ดีที่สุดจากการ train แล้ว เราจึงนำ model ที่ได้ไปทดสอบกับ test data set ประมาณ 160 รูป (8 classes  class ละ 20 รูป) ผลการทำนายมีความแม่นยำมากกว่า 90% เพื่อความแน่ใจว่า model ที่ได้มานั้นจะไม่ได้ overfit กับข้อมูลส่วนอื่นๆในภาพ เราจึงได้นำโปรแกรม GradCAM มาช่วยยืนยันว่าโมเดลนั้นใช้ข้อมูลจากส่วนไหนของรูปภาพเพื่อนำมาทำนาย ผลการยืนยันจากโปรแกรม GradCAM ทำให้เห็นได้ว่าผลการทำนายส่วนใหญ่นำข้อมูลจากรูปของผลไม้ไปใช้วิเคราะห์และทำนายจริง
ผลการทำนายจากtest dataset

![image](https://github.com/khwanck/DeepMyeCNN/blob/main/images/output1.png)

ผลการทำนายจาก GradCAM

![image](https://github.com/khwanck/DeepMyeCNN/blob/main/images/output2.png)
![image](https://github.com/khwanck/DeepMyeCNN/blob/main/images/output3.png)

## Discussion & Conclusion:
  เราสามารถนำโมเดลสำเร็จรูปที่มีการเทรนมาแล้วเป็นอย่างดีเช่น VGG16, ResNet, MobileNet, DenseNet มาเป็น Pre-training โดยทำการปิดส่วน Classification เดิม แล้วทำการเพิ่ม layer Classification ของเราขึ้นมาใหม่ได้เอง เพื่อนำมาใช้เทรนกับชุดข้อมูลที่เราสนใจและไม่มีอยู่ใน imagenet ได้  เนื่องจากชุดข้อมูลที่เรานำมาเทรนใหม่เป็นชุดข้อมูลเกี่ยวกับผลไม้ไทยซึ่งมีลักษณะ สีสัน แตกต่างกันพอสมควร จึงทำให้เราสามารถสร้างโมเดลมาทำนายได้อย่างแม่นยำโดยไม่ยากนัก  ถ้าเป็นชุดข้อมูลอื่นๆที่มีรูปภาพที่ซับซ้อนหรือคลุมเครืออาจจะทำให้ไม่สามารถทำนายได้ดีเช่นนี้
สามารถ modify GradCAM code มาใช้กับ code ในงานของกลุ่มได้และสามารถใช้ GradCAM ช่วยยืนยันว่าโมเดลทำนายจากข้อมูลผลไม้ในรูปนั้นจริง

## Issue:
*	เมื่อ Format ของรูปไม่ตรงตามที่ command python รู้จักทำให้ไม่สามารถอ่านรูปเข้ามาได้ จึงทำการเปลี่ยน format ของรูปเป็น JPEG ก่อนทุกรูปเพื่อง่ายต่อการอ่าน
*	ตอนแรกทำการ resize รูปทุกรูปให้มีขนาด 128x128 ก่อนเพื่อนำไป train แต่พอจะนำรูปที่ออกจาก conv2d layer สุดท้ายไปใช้ทำ GradCAM ขนาดของรูปที่ได้จาก conv2d layer สุดท้ายมีขนาดเล็กเกินไป (3x3) ทำให้ได้ GradCAM ที่ไม่ค่อยละเอียด เลยทำการเปลี่ยนขนาดรูปเป็น 256x256 แทนเพื่อให้ GradCAM มีความละเอียด (6x6) มากขึ้น อาจจะทำให้เสียเวลาเทรนขึ้นเล็กน้อยแต่ก็ทำให้ผลการทำนายแม่นขึ้นด้วย

## References:
* https://towardsdatascience.com/supercharge-image-classification-with-transfer-learning-4b8c52e69c0c
* https://blog.roboflow.com/how-to-train-mobilenetv2-on-a-custom-dataset/
* https://medium.datadriveninvestor.com/deep-learning-day-2-practical-implementation-of-cnn-9b51ba9d28a9
* https://www.pluralsight.com/guides/introduction-to-densenet-with-tensorflow
* https://keras.io/examples/vision/grad_cam/
* https://muthu.co/understanding-the-classification-report-in-sklearn/

## Team Members
* ทุกคนช่วยกันรวบรวมรูปของผลไม้แต่ละชนิด

ID   | Responsibility |% Contribute
--------- | ------ | ------
6310422062 | Experiment Pre-training model | 20%
6310422066 | Tuning parameters of model  | 20%
6310422067 | Experiment Augmentation | 20%
6310422068 | Cleaning Image format & Separate Data Set | 20%
6310422070 | Evaluate train method on test set | 20%

## End Credit
โปรเจคนี้เป็นส่วนหนึ่งของวิชา Deep Learning (BADS7604), หลักสูตรวิทยาศาสตรมหาบัณฑิต สาขาวิชาการวิเคราะห์ธุรกิจและวิทยาการข้อมูล  , สถาบันบัณฑิตพัฒนบริหารศาสตร์ (NIDA)
