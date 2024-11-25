# 460_RT-DETRv2-Wholebody25

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.10229410.svg)](https://doi.org/10.5281/zenodo.10229410)

This model far surpasses the performance of existing CNNs in both inference speed and accuracy. I'm not particularly interested in comparing performance between architectures, so I don't cherry-pick any of the verification results. What is important is a balance between accuracy, speed, the number of output classes, and versatility of output values.

Lightweight human detection models generated on high-quality human data sets. It can detect objects with high accuracy and speed in a total of 25 classes: `Body`, `Adult`, `Child`, `Male`, `Female`, `Body_with_Wheelchair`, `Body_with_Crutches`, `Head`, `Front`, `Right_Front`, `Right_Side`, `Right_Back`, `Back`, `Left_Back`, `Left_Side`, `Left_Front`, `Face`, `Eye`, `Nose`, `Mouth`, `Ear`, `Hand`, `Hand_Left`, `Hand_Right`, `Foot`. Even the classification problem is being attempted to be solved by object detection. There is no need to perform any complex affine transformations or other processing for pre-processing and post-processing of input images. In addition, the resistance to Motion Blur, Gaussian noise, contrast noise, backlighting, and halation is quite strong because it was trained only on images with added photometric noise for all images in the MS-COCO subset of the image set. In addition, about half of the image set was annotated by me with the aspect ratio of the original image substantially destroyed. I manually annotated all images in the dataset by myself. The model is intended to use real-world video for inference and has enhanced resistance to all kinds of noise. Probably stronger than any known model. However, the quality of the known data set and my data set are so different that an accurate comparison of accuracy is not possible.

The aim is to estimate head pose direction with minimal computational cost using only an object detection model, with an emphasis on practical aspects. The concept is significantly different from existing full-mesh type head direction estimation models, head direction estimation models with tweaked loss functions, and models that perform precise 360° 6D estimation. Capturing the features of every part of the body on a 2D surface makes it very easy to combine with other feature extraction processes.

Don't be ruled by the curse of mAP.

- Difficulty: Normal

  https://github.com/user-attachments/assets/646ab997-f901-4626-88fe-d274a12c9fda

- Difficulty: Normal

  https://www2.nhk.or.jp/archives/movies/?id=D0002160854_00000

  https://github.com/user-attachments/assets/5b47f128-4e27-4a71-bcb4-19507fe8be27

- Difficulty: Super Hard
  - The depression and elevation angles are quite large.
  - People move quickly. (Intense motion blur)
  - The image quality is quite poor and there is a lot of noise. (Quality taken around 1993)
  - The brightness is quite dark.

  https://www2.nhk.or.jp/archives/movies/?id=D0002080169_00000

  https://github.com/user-attachments/assets/bb19455d-8c3f-4bfa-abe8-143f16b93388

- Difficulty: Super Ultra Hard (Score threshold 0.35)
  - Heavy Rain.
  - High intensity halation.
  - People move quickly. (Intense motion blur)
  - The image quality is quite poor and there is a lot of noise. (Quality taken around 2003)
  - The brightness is quite dark.
  
  https://www2.nhk.or.jp/archives/movies/?id=D0002040195_00000

  https://github.com/user-attachments/assets/cd7037ff-dee1-4b63-ad5d-c838ee639218

- Difficulty: Super Hard (1600x898 -> 640x640)

  A major weakness of RT-DETR is that it cannot process anything other than its internal processing resolution of 640x640, so if an image with an unnecessarily large resolution is used, faces in the background will be scaled down to a size of less than one pixel. Therefore, if you use an image of an unnecessarily large size such as 1600x898, as in the image below, people in the rear will be almost impossible to detect. If you want accurate detection at higher resolutions, you will need to pre-train the model at a higher resolution setting, such as 1600x898.

  ![sample](https://github.com/user-attachments/assets/94aa6f51-2062-408a-89c7-714550fb92e4)

  The figure below shows the results of inference on the same image using a CNN with only 1MB. We can see that the performance of RT-DETRv2, which has an input resolution fixed at 640x640, is overwhelmingly lower than that of the 1MB CNN.

  Cited: https://github.com/biubug6/Face-Detector-1MB-with-landmark

  ![image](https://github.com/user-attachments/assets/314d3c85-6555-47b7-aaaa-2591c699167a)

  The detection results of YOLOv9-E that I created with the NMS limiter disabled are shown in the figure below.

  Cited: https://github.com/PINTO0309/PINTO_model_zoo/tree/main/459_YOLOv9-Wholebody25

  ![0009](https://github.com/user-attachments/assets/a14c08c9-49c7-41a5-bf9c-a049947e4c54)

  ![0007](https://github.com/user-attachments/assets/22405164-99de-4152-a5f3-a47088d54229)

- Difficulty: Normal (800x898 x2)

  Therefore, when using RT-DETRv2 and high-resolution images with aspect ratios that deviate significantly from 1:1, accuracy can be dramatically improved by simply dividing the images and performing inference in two batches so as to maintain the aspect ratio as much as possible. The figure below shows the results of inference in two batches, splitting the image into two parts, left and right, at 800x898 in size.

  |batch.1|batch.2|
  |:-:|:-:|
  |![sample_1](https://github.com/user-attachments/assets/dc86c0ec-3e01-4a69-80f2-8a1efa6ab041)|![sample_2](https://github.com/user-attachments/assets/c65049ec-bad2-4bef-bb38-25c4c9c473d1)|

  An even more important point to note is that the current 1,250-query RT-DETRv2 can only output bounding boxes for a maximum of 50 to 100 people. The image above probably contains around 300 people, so I would not be able to measure the true detection performance of Transformer unless I expanded it to 5,000 queries. After debugging, I found that 1,250 bounding boxes exceeded the score threshold, meaning that we were unable to output all objects that were within the range of our detection capability. This means that the system is ignoring objects that could have been detected and only outputting 1,250.

  ![image](https://github.com/user-attachments/assets/df4b93ff-30ed-4e77-baf9-8670a20bc807)

- Difficulty: Super Hard (1600x898 -> 640x640, 2,500 query)

  The results were a bit unexpected: when we generated a model with 2,500 queries and ran inference on the same images, the accuracy was actually significantly lower than when we ran the model with 1,250 queries. In other words, I can say the following two points.

  1. A large increase in the number of queries has a negative impact
  2. Keeping aspect ratios as close to 1:1 as possible maintains performance

  ![sample](https://github.com/user-attachments/assets/cc3e9349-6cbf-47bd-9263-0315d020faf5)

  Just to be safe, I have also included the results of inference performed by splitting the image into two halves, left and right. The accuracy is also clearly reduced here.

  |batch.1|batch.2|
  |:-:|:-:|
  |![sample_1](https://github.com/user-attachments/assets/67ec992b-e9c6-4677-b8db-ccde87e14961)|![sample_2](https://github.com/user-attachments/assets/5c7860fc-c810-441c-9fb2-1130eb595baf)|

- Difficulty: Super Hard (Score threshold 0.35)

  https://www.pakutaso.com/20240833234post-51997.html

  ![image](https://github.com/user-attachments/assets/a39a9ba7-ac3a-4199-bbd2-b6beed7b072b)

- Difficulty: Normal

  Cited: https://github.com/Kazuhito00/RT-DETR-ONNX-Sample

  |Pure MS-COCO trained|Self-annotated MS-COCO trained|
  |:-:|:-:|
  |![image](https://github.com/user-attachments/assets/0318bf9d-9815-40ea-885a-cdc7e526056d)|![image](https://github.com/user-attachments/assets/5293e3ad-cbc1-4e0d-bca0-7e894ab80988)|

- Other results

  |output<br>`Objects score threshold >= 0.65`<br>`Attributes score threshold >= 0.70`<br>`1,250 query`|output<br>`Objects score threshold >= 0.65`<br>`Attributes score threshold >= 0.70`<br>`1,250 query`|
  |:-:|:-:|
  |![image](https://github.com/user-attachments/assets/2b310a9f-1203-4db4-9dc8-2129532e3f0d)|![image](https://github.com/user-attachments/assets/c99fb457-a813-4792-b773-84787298a359)|
  |![image](https://github.com/user-attachments/assets/fe6df76e-ce43-49c9-af58-4340c4b9502e)|![image](https://github.com/user-attachments/assets/faf65954-3d4b-4d4c-93c1-9b2573a9858a)|
  |![image](https://github.com/user-attachments/assets/e2cbd298-6072-4e28-bd3f-b150a704d4af)|![image](https://github.com/user-attachments/assets/711b73f1-2863-4298-b646-2ad2d527b327)|
  |![image](https://github.com/user-attachments/assets/d6e66287-cb5b-4806-8bd0-3781750ada3b)|![image](https://github.com/user-attachments/assets/e1c7f9d1-8752-4fa0-8219-48f222846fc3)|
  |![image](https://github.com/user-attachments/assets/0063bdd0-1317-40e5-b44b-dd01468436c2)|![image](https://github.com/user-attachments/assets/c9ea9f7c-e0db-4884-9734-dc6b79db5d05)|
  |![image](https://github.com/user-attachments/assets/70c57a45-a8ce-47dc-9bdd-6cca10d1b16a)|![image](https://github.com/user-attachments/assets/19aefce8-61d7-4294-bf3d-865529cae228)|
  |![image](https://github.com/user-attachments/assets/258805da-6578-4b05-b3d9-67850a027a03)|![image](https://github.com/user-attachments/assets/5114a254-f410-4db7-a61c-2391b8ccfbfb)|
  |![image](https://github.com/user-attachments/assets/a2629dd0-73f3-4115-a198-32562b206a7a)|![image](https://github.com/user-attachments/assets/e4c4bd6d-dc33-4d44-9be1-55c57a113ad3)|

The use of [CD-COCO: Complex Distorted COCO database for Scene-Context-Aware computer vision](https://github.com/aymanbegh/cd-coco) has also greatly improved resistance to various types of noise.

- Global distortions
  - Noise
  - Contrast
  - Compression
  - Photorealistic Rain
  - Photorealistic Haze
  - Motion-Blur
  - Defocus-Blur
  - Backlight illumination
- Local distortions
  - Motion-Blur
  - Defocus-Blur
  - Backlight illumination

## 1. Dataset
  - COCO-Hand http://vision.cs.stonybrook.edu/~supreeth/COCO-Hand.zip
  - [CD-COCO: Complex Distorted COCO database for Scene-Context-Aware computer vision](https://github.com/aymanbegh/cd-coco)
  - I am adding my own enhancement data to COCO-Hand and re-annotating all images. In other words, only COCO images were cited and no annotation data were cited.
  - I have no plans to publish my own dataset.

## 2. Annotation

  Halfway compromises are never acceptable. I added `2,611` annotations to the following `480x360` image. The trick to annotation is to not miss a single object and not compromise on a single pixel. The ultimate methodology is to `try your best`.

  https://github.com/user-attachments/assets/32f150fe-ebf7-4374-9f0d-f1130badcfc1

  Please feel free to change the head direction label as you wish. There is no correlation between the model's behavior and the meaning of the label text.

  ![image](https://github.com/user-attachments/assets/765600a1-552d-4de9-afcc-663f6fcc1e9d) ![image](https://github.com/user-attachments/assets/15b7693a-5ffb-4c2b-9cc2-cc3022f858bb)

  |Class Name|Class ID|Remarks|
  |:-|-:|:-|
  |Body|0|Detection accuracy is higher than `Adult`, `Child`, `Male` and `Female` bounding boxes. It is the sum of `Adult`, `Child`, `Male`, and `Female`.|
  |Adult|1|Bounding box coordinates are shared with `Body`. It is defined as a subclass of `Body` as a superclass.|
  |Child|2|Bounding box coordinates are shared with `Body`. It is defined as a subclass of `Body` as a superclass.|
  |Male|3|Bounding box coordinates are shared with `Body`. It is defined as a subclass of `Body` as a superclass.|
  |Female|4|Bounding box coordinates are shared with `Body`. It is defined as a subclass of `Body` as a superclass.|
  |Body_with_Wheelchair|5||
  |Body_with_Crutches|6||
  |Head|7|Detection accuracy is higher than `Front`, `Right_Front`, `Right_Side`, `Right_Back`, `Back`, `Left_Back`, `Left_Side` and `Left_Front` bounding boxes. It is the sum of `Front`, `Right_Front`, `Right_Side`, `Right_Back`, `Back`, `Left_Back`, `Left_Side` and `Left_Front`.|
  |Front|8|Bounding box coordinates are shared with `Head`. It is defined as a subclass of `Head` as a superclass.|
  |Right_Front|9|Bounding box coordinates are shared with `Head`. It is defined as a subclass of `Head` as a superclass.|
  |Right_Side|10|Bounding box coordinates are shared with `Head`. It is defined as a subclass of `Head` as a superclass.|
  |Right_Back|11|Bounding box coordinates are shared with `Head`. It is defined as a subclass of `Head` as a superclass.|
  |Back|12|Bounding box coordinates are shared with `Head`. It is defined as a subclass of `Head` as a superclass.|
  |Left_Back|13|Bounding box coordinates are shared with `Head`. It is defined as a subclass of `Head` as a superclass.|
  |Left_Side|14|Bounding box coordinates are shared with `Head`. It is defined as a subclass of `Head` as a superclass.|
  |Left_Front|15|Bounding box coordinates are shared with `Head`. It is defined as a subclass of `Head` as a superclass.|
  |Face|16||
  |Eye|17||
  |Nose|18||
  |Mouth|19||
  |Ear|20||
  |Hand|21|Detection accuracy is higher than `Hand_Left` and `Hand_Right` bounding boxes. It is the sum of `Hand_Left`, and `Hand_Right`.|
  |Hand_Left|22|Bounding box coordinates are shared with `Hand`. It is defined as a subclass of `Hand` as a superclass.|
  |Hand_Right|23|Bounding box coordinates are shared with `Hand`. It is defined as a subclass of `Hand` as a superclass.|
  |Foot (Feet)|24||

  ![image](https://github.com/user-attachments/assets/49f9cbf3-3a9c-4666-84ae-d86148c34866)

## 3. Test
  - RTX3070 (VRAM: 8GB)
  - Python 3.10
  - onnx 1.16.1+
  - onnxruntime-gpu v1.18.1 (TensorRT Execution Provider Enabled Binary. See: [onnxruntime-gpu v1.18.1 + CUDA 12.5 + TensorRT 10.2.0 build (RTX3070)](https://zenn.dev/pinto0309/scraps/801db283883c38)
  - opencv-contrib-python 4.10.0.84+
  - numpy 1.24.3
  - TensorRT 10.2.0.19-1+cuda12.5

    ```bash
    # Common ############################################
    pip install opencv-contrib-python numpy onnx

    # For ONNX ##########################################
    pip uninstall onnxruntime onnxruntime-gpu

    pip install onnxruntime
    or
    pip install onnxruntime-gpu
    ```

  - Demonstration of models with built-in post-processing (Float32/Float16)
  - `score_threshold` is a very rough value set for testing purposes, so feel free to adjust it to your liking. The default threshold is probably too low.
  - There is a lot of information being rendered into the image, so if you want to compare performance with other models it is best to run the demo with `-dnm`, `-dgm`, `-dlr` and `-dhm`.

    ```
    usage:
      demo_rtdetrv2_onnx_wholebody25.py \
      [-h] \
      [-m MODEL] \
      (-v VIDEO | -i IMAGES_DIR) \
      [-ep {cpu,cuda,tensorrt}] \
      [-it] \
      [-ost] \
      [-ast] \
      [-dvw] \
      [-dwk] \
      [-dnm] \
      [-dgm] \
      [-dlr] \
      [-dhm] \
      [-oyt] \
      [-bblw]

    options:
      -h, --help
        show this help message and exit
      -m MODEL, --model MODEL
        ONNX/TFLite file path for RT-DETRv2-Wholebody25.
      -v VIDEO, --video VIDEO
        Video file path or camera index.
      -i IMAGES_DIR, --images_dir IMAGES_DIR
        jpg, png images folder path.
      -ep {cpu,cuda,tensorrt}, \
          --execution_provider {cpu,cuda,tensorrt}
        Execution provider for ONNXRuntime.
      -it {fp16,int8}, --inference_type {fp16,int8}
        Inference type. Default: fp16
      -ost OBJECT_SCORE_THRESHOLD, --object_score_threshold OBJECT_SCORE_THRESHOLD
        Object score threshold. Default: 0.65
      -ast ATTRIBUTE_SCORE_THRESHOLD, --attribute_score_threshold ATTRIBUTE_SCORE_THRESHOLD
        Attribute score threshold. Default: 0.70
      -dvw, --disable_video_writer
        Disable video writer. Eliminates the file I/O load associated with automatic
        recording to MP4. Devices that use a MicroSD card or similar for main
        storage can speed up overall processing.
      -dwk, --disable_waitKey
        Disable cv2.waitKey(). When you want to process a batch of still images,
        disable key-input wait and process them continuously.
      -dnm, --disable_generation_identification_mode
        Disable generation identification mode.
        (Press N on the keyboard to switch modes)
      -dgm, --disable_gender_identification_mode
        Disable gender identification mode.
        (Press G on the keyboard to switch modes)
      -dlr, --disable_left_and_right_hand_identification_mode
        Disable left and right hand identification mode.
        (Press H on the keyboard to switch modes)
      -dhm, --disable_headpose_identification_mode
        Disable HeadPose identification mode.
        (Press P on the keyboard to switch modes)
      -oyt, --output_yolo_format_text
        Output YOLO format texts and images.
      -bblw BOUNDING_BOX_LINE_WIDTH, --bounding_box_line_width BOUNDING_BOX_LINE_WIDTH
        Bounding box line width. Default: 2
    ```

- RT-DETRv2-Wholebody25 - S (rtdetrv2_r18vd_120e_wholebody25) - 1,250 query
  ```
  Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.602
  Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.802
  Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.653
  Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.432
  Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.731
  Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.867
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.330
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.615
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.694
  Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.553
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.820
  Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.924
  ```
- RT-DETRv2-Wholebody25 - X (rtdetrv2_r101vd_6x_wholebody25) - 1,250 query
  ```
  Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.650
  Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.841
  Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.700
  Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.498
  Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.769
  Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.899
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.346
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.647
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.727
  Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.598
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.847
  Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.938
  ```
- RT-DETRv2-Wholebody25 - X (rtdetrv2_r101vd_6x_wholebody25) - 2,500 query
  ```
  Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.621
  Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.820
  Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.668
  Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.454
  Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.757
  Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.894
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.341
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.627
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.713
  Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.580
  Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.839
  Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.941
  ```

- Pre-Process

  To ensure fair benchmark comparisons with YOLOX, `BGR to RGB conversion processing` and `normalization by division by 255.0` are added to the model input section.

  ![image](https://github.com/user-attachments/assets/9098f12d-6b04-497f-947e-02bc77855f51)

## 4. Citiation
  If this work has contributed in any way to your research or business, I would be happy to be cited in your literature.
  ```bibtex
  @software{RT-DETRv2-Wholebody25,
    author={Katsuya Hyodo},
    title={Lightweight human detection models generated on high-quality human data sets. It can detect objects with high accuracy and speed in a total of 25 classes: Body, Adult, Child, Male, Female, Body_with_Wheelchair, Body_with_Crutches, Head, Front, Right_Front, Right_Side, Right_Back, Back, Left_Back, Left_Side, Left_Front, Face, Eye, Nose, Mouth, Ear, Hand, Hand_Left, Hand_Right, Foot.},
    url={https://github.com/PINTO0309/PINTO_model_zoo/tree/main/460_RT-DETRv2-Wholebody25},
    year={2024},
    month={10},
    doi={10.5281/zenodo.10229410}
  }
  ```

## 5. Cited
  I am very grateful for their excellent work.
  - COCO-Hand

    https://vision.cs.stonybrook.edu/~supreeth/

    ```bibtex
    @article{Hand-CNN,
      title={Contextual Attention for Hand Detection in the Wild},
      author={Supreeth Narasimhaswamy and Zhengwei Wei and Yang Wang and Justin Zhang and Minh Hoai},
      booktitle={International Conference on Computer Vision (ICCV)},
      year={2019},
      url={https://arxiv.org/pdf/1904.04882.pdf}
    }
    ```

  - [CD-COCO: Complex Distorted COCO database for Scene-Context-Aware computer vision](https://github.com/aymanbegh/cd-coco)

    ![image](https://github.com/PINTO0309/PINTO_model_zoo/assets/33194443/69603b9b-ab9f-455c-a9c9-c818edc41dba)
    ```bibtex
    @INPROCEEDINGS{10323035,
      author={Beghdadi, Ayman and Beghdadi, Azeddine and Mallem, Malik and Beji, Lotfi and Cheikh, Faouzi Alaya},
      booktitle={2023 11th European Workshop on Visual Information Processing (EUVIP)},
      title={CD-COCO: A Versatile Complex Distorted COCO Database for Scene-Context-Aware Computer Vision},
      year={2023},
      volume={},
      number={},
      pages={1-6},
      doi={10.1109/EUVIP58404.2023.10323035}
    }
    ```

  - RT-DETRv2

    https://github.com/lyuwenyu/RT-DETR

    ```bibtex
    @misc{lv2024rtdetrv2improvedbaselinebagoffreebies,
          title={RT-DETRv2: Improved Baseline with Bag-of-Freebies for Real-Time Detection Transformer},
          author={Wenyu Lv and Yian Zhao and Qinyao Chang and Kui Huang and Guanzhong Wang and Yi Liu},
          year={2024},
          eprint={2407.17140},
          archivePrefix={arXiv},
          primaryClass={cs.CV},
          url={https://arxiv.org/abs/2407.17140},
    }
    ```

  - PINTO Custom RT-DETRv2 (Drastically change the training parameters and optimize the model structure)

    https://github.com/PINTO0309/RT-DETR

## 6. License
[Apache2.0](https://github.com/PINTO0309/PINTO_model_zoo/blob/main/460_RT-DETRv2-Wholebody25/LICENSE)

## 7. Next challenge
- `shoulder`, `elbow`, `knee`
- I would like to verify the hypothesis that the correlation between the positions of each part is embedded as weights in the CNN and Transformer.
- Therefore, we reduce the 2D visible information in the area enclosed by the annotation label box to the limit and investigate how the model behaves when only the 3x3 pixel label box is annotated.

  ![image](https://github.com/user-attachments/assets/25675c7a-b20b-48ba-93fe-f24d54b7cf2b)

- A state of provisional implementation

  https://github.com/user-attachments/assets/38c55669-2acf-46b4-a9c4-819349880854

