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

## Slide 8  RAG chooses a response path, then checks the plan    1:05

The advisory workflow uses retrieval-augmented generation, or RAG. It first retrieves relevant agricultural material through both dense vector search and BM25 keyword search. Reciprocal rank fusion combines those rankings, and a reranker selects the most relevant passages. A router then chooses a path according to the question: a direct factual answer, a deeper answer that can extend retrieval to web sources, or a treatment-planning path.

For treatment planning, the system brings together retrieved knowledge, the visual diagnosis, and current sensor readings. It drafts a structured response with potential measures, application steps, schedules, and precautions. Before a plan is returned, a safety-audit node checks specified rules such as pesticide dosage and pre-harvest interval; a refinement node can revise flagged content.

The retrieval algorithm shown near the start supplies the knowledge for these response paths. The next two slides explain treatment planning and the safety audit. These components describe an implemented workflow. The paper does not report quantitative retrieval scores, answer-faithfulness scores, or an agronomist assessment of audit effectiveness. Those evaluations are essential before treating generated plans as validated field guidance.

## Slide 2  Hybrid retrieval combines two searches    0:45

Before the overview, this slide highlights one technical building block of Leafy: hybrid retrieval. Algorithm 1 retrieves twenty candidates with dense vector search and twenty with BM25 keyword search. Dense search finds material with related meaning, while BM25 matches query terms. Reciprocal rank fusion combines the rankings and returns the top ten documents, along with identifiers of the dense matches. A later reranker selects passages for generation. This is the implemented retrieval procedure; its quality has not yet been quantitatively benchmarked. I will place this component within the full advisory workflow shortly.

## Slide 9  Treatment planning uses farm context    0:45

Algorithm 3 shows the planning gate. If the best score among retrieved documents and web results falls below 0.7, the planner returns “cannot plan.” Otherwise it builds context from those sources, the current environmental readings, and any safety issues returned during refinement. The language model then generates a structured plan. The score threshold is part of the algorithm, not evidence that the resulting advice is agronomically validated.

## Slide 10  The safety audit checks the draft plan    0:50

Algorithm 4 examines a generated plan before delivery. It applies hard checks against the plan, environmental state, and banned list. Depending on the outcome, it can pass the plan or return issues for refinement. An additional language-model audit checks the plan against safety rules. The implementation targets errors such as dosage and pre-harvest interval violations, but the paper does not quantify how reliably this node detects or corrects them.

## Slide 11  The evaluation separates model tasks from integration    1:05

The study evaluates the two visual models according to their own tasks and then reports functional integration of the wider system. The slide identifies the image source as the Kaggle dataset organized-coffee-leaf-diseases, compiled from RoCoLe and other open sources. It covers Healthy, Leaf Miner, Phoma Leaf Spot, Red Spider Mite, and Coffee Leaf Rust. For classification, the study uses a balanced evaluation set of 806 images and repeats MobileNetV2 training under five random seeds. The paper describes this set as both its validation and final test set, so it is not an independent external farm test.

For leaf localization, YOLOv8n was trained for 50 epochs on annotated leaf regions and evaluated with mean average precision. The IoT and RAG components were assessed through operation of the implemented workflows, rather than through controlled reliability or answer-quality benchmarks. This separation matters: the measured classifier and detector metrics do not automatically establish performance for the complete diagnosis-to-advice chain.

## Slide 12  Classification performance varies across runs    1:20

This chart summarizes the five MobileNetV2 accuracy results before we look at the selected run’s training curves and confusion matrix. Seed 42 reached 95.04 percent; the other runs reached 94.04, 92.93, 92.93, and 94.17 percent. The mean was 93.82 percent with a standard deviation of 0.94 percentage points. The selected seed-42 run also achieved 95.18 percent macro F1, while the five-run mean macro F1 was 93.74 percent.

The training used ten epochs for the classification head followed by fifteen epochs of fine-tuning. Repeating the run helps show that the result is not entirely dependent on one initialization. At the same time, a spread of more than two percentage points between the highest and lowest accuracy warns us against presenting only the best run as typical performance. All of these figures come from the same balanced 806-image evaluation set. Performance under new farms, devices, lighting, and symptom stages remains to be established.

## Slide 13  MobileNetV2 training across two stages    0:55

This figure shows the selected seed-42 run over twenty-five epochs. The dashed line separates ten epochs of head training from fifteen epochs of fine-tuning. Validation accuracy and precision dip briefly when fine-tuning begins, then recover. By epoch twenty-five, validation accuracy reaches 95.04 percent. The curves describe optimization on this study’s validation set. Because that same 806-image set also served as the final test set, this plot does not establish performance on an independent farm dataset.

## Slide 14  Rust has the lowest recall    1:00

The confusion matrix gives the underlying counts for the selected run. Read each row as the actual class and each column as the predicted class. For Rust, the model correctly classified 158 of 180 images. It assigned the remaining 22 to another class, including 11 labelled Healthy. That produces 87.78 percent Rust recall, lower than the other four classes. The matrix makes the error pattern more concrete than overall accuracy alone. These are results on the shared validation and test set, so field performance remains unknown.

## Slide 15  YOLOv8n leaf localization over 50 epochs    1:00

The YOLOv8n figure shows falling training losses and improving localization metrics over fifty epochs. A dashed marker at epoch forty shows where mosaic augmentation was disabled. The detector reached 97.9 percent mAP at an intersection-over-union threshold of 0.50 at epoch forty-six. The stricter mAP averaged across thresholds from 0.50 to 0.95 reached 83.3 percent at epoch fifty. Those peaks occur at different checkpoints. They measure leaf boxes, not disease classification.

## Slide 16  What the prototype establishes    1:15

The implemented system demonstrates several useful operations. On-device TensorFlow Lite inference returned a preliminary two-stage diagnosis in approximately 400 milliseconds. With connectivity, the cloud path provided a more detailed diagnosis and stored it in crop history. The physical sensor node sent readings through MQTT to backend processing and PostgreSQL; the readings appeared in dashboards, and threshold events produced push notifications.

Those observations show that the components can work together. They are not yet a full reliability or deployment benchmark. The paper does not give latency distributions across different phones or quantify the effect of TensorFlow Lite quantization. It does not measure MQTT packet delivery, reconnection, or message recovery under deliberately degraded plantation networks. Likewise, the RAG routes and safety-audit step were implemented, but their quality and effectiveness were not quantitatively scored. This boundary is important when considering field deployment.

## Slide 17  The contribution is integration; validation comes next    1:15

Leafy’s contribution is an integrated path from a coffee leaf image and measured farm conditions to a contextual advisory draft. The detector and classifier provide promising measured results on the study data, and the physical prototype shows that sensing, storage, dashboards, alerts, and advisory routing can operate within one platform.

The next step is broader validation. For vision, that means independent farms and image-capture devices, early symptom stages, and leaves with more than one condition. For sensing, it means calibration against reference instruments, outdoor durability, power profiling, and controlled tests of degraded connectivity. For advice, it means measuring retrieval and answer quality, testing the safety audit against known errors, and asking agronomists to assess the plans. A controlled end-to-end study is also needed to establish whether the combined workflow improves decisions in practice.

## Slide 18  THANK YOU    0:25

Thank you for your attention. Leafy offers a working foundation for combining edge vision, environmental sensing, and knowledge-guided advice in coffee farming. I welcome your questions about the model results, the physical prototype, and the validation needed before wider field use.
