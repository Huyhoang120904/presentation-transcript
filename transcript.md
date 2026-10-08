# Leafy Presenter Transcript

ICATSD 2026 · 18 slides · Approximately 17 minutes

This script follows the current Leafy presentation in its exact slide order and uses the paper for technical context. Timings are guides; allow a brief pause on the algorithm and results figures.

## Slide 1  LEAFY    0:45

Good morning, everyone. Today I’m presenting Leafy, our framework for intelligent coffee crop monitoring and decision support. The work brings together three capabilities: visual diagnosis on an edge device, environmental monitoring from a physical sensor node, and an advisory workflow that draws on agricultural knowledge and the farm’s current conditions.

I’ll explain how the system is built, show the results we measured for leaf localization and disease classification, and distinguish those measurements from the parts of the prototype that we verified functionally. The main question throughout is how to move from a disease label toward a more useful field decision.

## Slide 3  CONTENTS    0:20

The presentation covers the introduction and research gap, the literature behind the approach, our methodology, the measured results, and the discussion of limitations and future work. I will begin with the farmer’s problem, then show how vision, sensing, and retrieval fit into the platform.

## Slide 4  A diagnosis is only the first step    1:00

Coffee leaf rust, leaf miners, Phoma leaf spot, and red spider mites can threaten crop quality and yield. For smallholder farmers, timely expert inspection may be expensive or difficult to arrange across dispersed farms. Computer vision can offer a faster first indication, but a label alone rarely answers the farmer’s next question: what should I do here, under these conditions?

That is the gap motivating Leafy. Existing work has advanced image recognition, low-cost sensing, and agricultural advisory systems, often as separate components. We connect them so that an image can identify a possible condition, sensors can describe the environment around the crop, and a knowledge-guided workflow can prepare a contextual response. The goal is decision support for the farmer, with the limits of the current evidence kept visible.

## Slide 5  One platform, two sources of context    1:05

The system begins with two sources of context. The first is the leaf image. A detector identifies the leaf region, and a classifier assigns a condition to the cropped image. The second is telemetry from the farm: air temperature and humidity, soil moisture, and light intensity. These measurements are stored and can be shown in dashboards or used for threshold alerts.

The advisory component then combines a visual diagnosis, available sensor readings, and retrieved agricultural material. It can produce a structured treatment draft and pass that draft through a safety check before delivery. Behind this simple view is a microservices platform that handles client requests, storage, messaging, notifications, and retrieval. The key architectural idea is that the advisory stage receives both a symptom signal and a measured environmental context.

## Slide 6  The vision pipeline removes background clutter    1:00

A field photograph often contains branches, soil, other leaves, and variable lighting. Leafy therefore uses a two-stage vision pipeline. First, YOLOv8n locates leaf regions in the image. The highest-confidence box is selected, and that region is cropped to reduce background clutter. Second, the crop is resized to 224 by 224 pixels and passed to MobileNetV2 for a single label: Healthy, Miner, Phoma, Red Spider Mite, or Rust.

Both models are exported to TensorFlow Lite for a preliminary diagnosis on a mobile device, including when connectivity is unavailable. The cloud service runs the corresponding pipeline through FastAPI, provides more detailed results, and saves the diagnosis to crop history. The models have different jobs: detection evaluates where the leaf is; classification evaluates which condition it shows. Their results should therefore be read separately.

## Slide 7  A physical node supplies the farm context    1:05

Leafy’s physical node is based on an ESP32-CAM. A DHT11 measures air temperature and humidity, while a YL-69 sensor measures soil moisture and an LDR measures light. The ADS1115 converts the two analog sensor signals. Firmware averages samples and validates readings before transmission.

The node publishes telemetry through MQTT to the backend. The system stores raw data in PostgreSQL, keeps a current snapshot, and builds five-minute, hourly, and daily aggregates for the web and mobile dashboards. When a reading crosses a configured threshold, the platform can generate an alert and send a push notification. These readings can also enter the advisory context. The paper verifies this complete flow with a physical prototype, while leaving long-term calibration and weak-network reliability for future testing.

##  Slide 9 — RAG System Pipeline (0:45)

The RAG workflow has four main stages: retrieve, route, plan, and audit.

First, hybrid retrieval combines dense vector search and BM25 keyword matching. Reciprocal rank fusion and reranking improve the ordering of retrieved passages.

Next, a router selects one of three response paths: a fast factual answer, a deeper response that may include web retrieval, or treatment planning.

The planning path combines retrieved knowledge with the diagnosis and sensor readings.

Finally, a safety audit checks the draft before it is returned.

The following slides highlight three important algorithms in this workflow.

## Slide 10 — Hybrid Retrieval (0:30)

Algorithm 1 describes our hybrid retrieval strategy.

Dense search retrieves the top twenty semantically related documents, while BM25 retrieves twenty keyword-based matches.

Reciprocal rank fusion, or RRF, combines the two rankings and returns the top ten results.

A subsequent reranker selects the most relevant passages for generation.

This procedure is implemented, although retrieval quality has not yet been quantitatively evaluated.

## Slide 11 — Context-Aware Treatment Planning (0:35)

Algorithm 3 describes how the system generates treatment plans.

First, it checks the relevance of the retrieved information. If the best source score is below 0.7, the planner returns a cannot-plan result.

Otherwise, it combines retrieved documents, web results, current environmental readings, and any feedback from previous safety checks.

The language model then generates a structured plan containing treatment measures, application steps, schedules, and precautions.

This relevance threshold acts as a planning gate, rather than a guarantee of advice quality.

## Slide 12 — Safety Audit (0:35)

Algorithm 4 introduces a safety-audit stage for generated treatment plans.

The system first performs rule-based checks using the draft plan, environmental information, and predefined restrictions.

An additional language-model audit examines compliance with safety rules, including pesticide dosage and pre-harvest intervals.

If issues are detected, the plan can be returned for refinement.

This mechanism provides an additional screening layer, but its effectiveness still requires quantitative evaluation and expert validation.

## Slide 13 — Experimental Setup (0:50)

We now move to experimental evaluation.

The image dataset was obtained from Kaggle and compiled from RoCoLe and other open sources. It contains five categories: Healthy, Leaf Miner, Phoma Leaf Spot, Red Spider Mite, and Coffee Leaf Rust.

For classification, we used a balanced evaluation set of 806 images and repeated MobileNetV2 training with five random seeds.

YOLOv8n was trained separately for leaf localization.

The IoT and RAG components were evaluated through functional integration rather than controlled quantitative benchmarks.

An important limitation is that the 806-image evaluation set also served as the reported validation and test set. Independent field validation remains necessary.

## Slide 14 — MobileNetV2 Classification Results (0:50)

This chart compares MobileNetV2 performance across five training runs.

The model achieved a mean accuracy of 93.82 percent, with a standard deviation of 0.94 percentage points.

The best run, using seed 42, reached 95.04 percent accuracy and 95.18 percent macro F1.

Accuracy across the five runs ranged from 92.93 to 95.04 percent.

These results suggest relatively consistent performance across different initializations.

However, they were obtained from the same evaluation dataset, so they do not establish generalization to new farms or image-capture conditions.

## Slide 15 — MobileNetV2 Training Curves (0:35)

This figure shows the selected model's training process.

Training was divided into two stages: ten epochs for the classification head, followed by fifteen epochs of fine-tuning.

We can observe a brief decrease in validation performance when fine-tuning begins, followed by recovery and improvement.

By epoch twenty-five, validation accuracy reached 95.04 percent.

These curves illustrate the optimization process, while independent external evaluation remains future work.

## Slide 16 — Confusion Matrix (0:40)

The confusion matrix provides a more detailed view of classification errors.

For the selected run, Coffee Leaf Rust has the lowest recall among the five classes.

The model correctly classified 158 out of 180 Rust images, corresponding to 87.78 percent recall.

Of the twenty-two misclassified Rust images, eleven were predicted as Healthy.

This is particularly important because missed disease cases could affect subsequent treatment decisions.

Therefore, improving difficult-class recognition remains a priority.

## Slide 17 — YOLOv8n Detection Results (0:40)

Next, we evaluate the leaf-localization model.

YOLOv8n was trained for fifty epochs, with training losses generally decreasing and detection performance improving.

At epoch forty-six, the model achieved 97.9 percent mAP at IoU 0.50.

At epoch fifty, the stricter mAP averaged from IoU 0.50 to 0.95 reached 83.3 percent.

These results demonstrate promising performance for leaf localization.

However, these are detection metrics and should not be interpreted as disease-classification accuracy.

## Slide 18 — Results and Future Work (0:55)

Beyond the individual models, we also tested the integrated prototype.

The on-device TensorFlow Lite pipeline provided preliminary diagnosis in approximately 400 milliseconds, supporting offline operation.

The physical IoT node successfully transmitted measurements through MQTT. Data appeared on dashboards, and configured threshold events triggered push notifications.

These observations demonstrate functional integration across the platform.

However, several aspects remain unbenchmarked, including inference latency across different devices, sensor communication reliability under weak networks, and the quality of RAG-generated advice.

These are important next steps before wider deployment.

## Slide 19 — Conclusion and Validation Priorities (0:55)

To conclude, Leafy's main contribution is the integration of edge AI, environmental sensing, and retrieval-augmented decision support into one framework.

The vision models achieved promising results on the study dataset, while the physical prototype demonstrated working sensing, monitoring, notification, and advisory workflows.

Our next priorities are divided into three areas.

For vision, we need independent field datasets and more complex disease conditions.

For IoT, we need sensor calibration, long-term durability testing, and network reliability evaluation.

For RAG, we need retrieval and answer-quality benchmarks, safety-audit testing, and assessment by agricultural experts.

These evaluations are necessary to establish the system's practical reliability.

## Slide 20 — Thank You (0:15)

Thank you for your attention.

We hope Leafy provides a useful foundation for more intelligent and context-aware coffee farming.

I welcome your questions.
