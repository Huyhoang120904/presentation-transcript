# Leafy Research Presentation Transcript

**Target speaking time:** approximately 12 minutes  
**Deck:** `Leafy-Research-Presentation-12min.pptx`

## Slide 1 — Leafy

Good morning or good afternoon. We are Nguyễn Huy Hoàng and Lê Ngọc Hảo from the Faculty of Information Technology at the Industrial University of Ho Chi Minh City. Under the supervision of Dr. Tôn Long Phước, we developed Leafy, an AI, IoT, and retrieval-augmented generation platform for coffee leaf disease monitoring and diagnosis. This presentation focuses on the research problem, the proposed architecture, our experimental results, and the limitations that guide our next stage of development.

## Slide 2 — Problem and research gap

Coffee leaf diseases such as rust, leaf miner, Phoma leaf spot, and red spider mite threaten crop health and yield. Farmers often depend on visual experience or visits from agricultural experts, which can be slow, costly, and difficult to scale across remote growing areas. Existing research has produced strong image classifiers, but three gaps remain. Many systems return only a disease label rather than an actionable treatment plan. Edge deployment and cloud refinement are rarely combined in one diagnostic pipeline. Environmental monitoring, image diagnosis, and evidence-based advice also tend to exist as separate systems. Leafy addresses these gaps in one platform designed for field use.

## Slide 3 — Research contributions

Our work makes four contributions. First, we developed a two-stage vision pipeline that uses YOLOv8n to localize the leaf and MobileNetV2 to classify its condition. The same workflow operates on mobile devices through TensorFlow Lite and in the cloud through FastAPI. Second, we connected a LangGraph RAG pipeline to retrieved agricultural knowledge, diagnosis results, and current farm sensor data. Third, we implemented an IoT monitoring and alert workflow from the ESP32-CAM node to the mobile user. Finally, we organized the platform as twelve Java and Python microservices with Kafka-based asynchronous communication, allowing major workloads to operate and scale independently.

## Slide 4 — Leafy system architecture

The architecture has four main layers. Farmers use the React Native mobile application or the React web portal. All external requests pass through Spring Cloud Gateway, which handles JWT validation, rate limiting, and routing. Core business services run mainly on Java 21 and Spring Boot, while disease detection and RAG use Python and FastAPI. Services communicate directly when an immediate response is required and use Kafka for asynchronous events. PostgreSQL stores IoT time-series data, while MongoDB, Redis, Qdrant, and Elasticsearch support other workloads. Eureka provides service discovery, and Firebase delivers push notifications. This separation lets us scale AI, messaging, and monitoring components according to their own demand.

## Slide 5 — IoT monitoring and alerts

The IoT node uses an ESP32-CAM with a DHT11 for air temperature and humidity, a YL-69 for soil moisture, and an LDR for relative light intensity. An ADS1115 converter reads the analog sensors with better resolution and stability. The firmware validates and averages readings before sending telemetry through MQTT. The backend stores raw measurements, updates the latest snapshot, and aggregates data into five-minute, hourly, and daily views. It also evaluates user-defined thresholds. When a reading exceeds a threshold, the service creates an alert and applies a cooldown to prevent duplicate notifications. Kafka then carries the event to the notification service, which sends a push message through Firebase Cloud Messaging.

## Slide 6 — Two-stage diagnostic pipeline

Field photographs contain branches, soil, shadows, and other background noise. Direct classification can therefore learn irrelevant features. Our first stage uses YOLOv8n to detect leaf regions and return bounding boxes with confidence scores. The user or system selects the highest-confidence leaf crop. The second stage resizes that region to 224 by 224 pixels and passes it to MobileNetV2, which predicts one of five classes: Healthy, Miner, Phoma, Red Spider Mite, or Rust. Both models are converted to TensorFlow Lite for mobile inference. The original image can later upload to the cloud for detailed processing and storage. This design provides an immediate preliminary result even when connectivity is unavailable.

## Slide 7 — RAG treatment planning

The advisory service uses a multi-stage LangGraph workflow rather than a single prompt. It first compresses a long conversation when necessary. Hybrid retrieval combines dense semantic search and BM25 keyword search in Qdrant. A CrossEncoder reranks the retrieved documents. A router then selects a fast answer, a deeper response, or a structured treatment-plan path. The planning path can add trusted web evidence and the latest environmental readings from the farm. Before the user receives a plan, a safety auditor checks pesticide dosage and pre-harvest interval requirements. If it finds a problem, the refinement loop revises the draft before delivery. This process converts a diagnosis into evidence-grounded actions and a care schedule.

## Slide 8 — Dataset and experimental setup

For classification, we used the Organized Coffee Leaf Diseases dataset, compiled from RoCoLe and other open sources. It contains five balanced classes. The training set contains 3,227 images, and the test set contains 806 images. We trained MobileNetV2 in two phases: ten epochs with the base frozen, followed by fifteen fine-tuning epochs using AdamW. For leaf detection, we used approximately 1,500 annotated images and trained YOLOv8n for 50 epochs with 640-by-640 inputs. Data augmentation included changes in orientation, scale, brightness, and contrast. Training ran on Google Colab with GPU acceleration and mixed-precision computation.

## Slide 9 — AI model results

The detector achieved an mAP at an intersection-over-union threshold of 0.5 of 97.8 percent. Its stricter mAP across thresholds from 0.5 to 0.95 reached 83.3 percent. MobileNetV2 reached 92.80 percent validation accuracy and a macro F1-score of 0.93 on the five-class task. Phoma achieved the strongest recall at 1.00. Red Spider Mite had the lowest recall at 0.83 because its surface symptoms can resemble old leaves or nutrient deficiencies. During fine-tuning, replacing Adam with AdamW improved validation accuracy by 4.21 percentage points and reduced validation loss. These results support the two-stage design while identifying the class that needs more representative data.

## Slide 10 — Integrated application workflow

These interfaces show how the research components connect in the working platform. A farmer can capture or upload an image and see the detected leaf region. Local TensorFlow Lite inference returns the preliminary two-stage result in approximately 400 milliseconds. When online, the system synchronizes the image and diagnosis with the crop history. From that result, the farmer can request an RAG-generated treatment plan, review its supporting sources, and add its actions to a calendar. The same platform displays current environmental measurements and sends alerts when configured thresholds are exceeded. The implementation therefore links sensing, diagnosis, advice, and follow-up within one user workflow.

## Slide 11 — Limitations and future work

The prototype has several limitations. Most images come from international open datasets rather than Vietnamese coffee farms, so changes in lighting, background, disease stage, and local cultivation conditions may reduce field accuracy. Red Spider Mite recall remains the weakest. The IoT evaluation confirms data-flow completeness, but long-term physical deployment is still required to measure calibration error, corrosion, connectivity, and device durability. Deep RAG paths currently take about three to six seconds. The vision pipeline classifies a disease but does not measure the damaged area. Our next work will collect regional data, test the platform on real farms, add severity segmentation, improve RAG evaluation and speed, and investigate automated irrigation or precision spraying.

## Slide 12 — Conclusion and Q&A

In conclusion, Leafy demonstrates that edge AI, cloud services, IoT monitoring, and retrieval-augmented generation can operate as one practical system for coffee plant health. The detector achieved 97.8 percent mAP at 0.5, the classifier reached 92.80 percent validation accuracy with a macro F1-score of 0.93, and mobile inference provided a result in approximately 400 milliseconds. The current evidence supports the technical feasibility of the integrated workflow, while regional data and long-term field validation remain essential before large-scale deployment. Thank you for your attention. We welcome your questions.

## Likely questions and short answers

### Why use two models?

YOLOv8n removes background noise by localizing the leaf. MobileNetV2 then concentrates on disease features in the cropped region. This improves robustness for field images and remains light enough for mobile execution.

### Why choose MobileNetV2?

MobileNetV2 balances accuracy, model size, and computational cost. Its lightweight architecture supports TensorFlow Lite inference on a mobile device.

### Can the system work offline?

The mobile application can run both quantized vision models locally for a preliminary diagnosis. Cloud synchronization, the full RAG service, and centralized history require connectivity.

### How do you reduce unsafe RAG advice?

The pipeline retrieves supporting evidence, includes a dedicated safety-audit node, and checks pesticide dosage and pre-harvest interval requirements before returning a treatment plan.

### What is the weakest experimental result?

Red Spider Mite recall is 0.83, the lowest of the five classes. The next step is to collect more representative images and investigate techniques such as focal loss or hard-example mining.

### Has the IoT hardware been tested on farms for a long period?

No. The current work validates the prototype data flow and system integration. Long-term field trials are still needed to evaluate sensor calibration, corrosion, network reliability, and durability.
