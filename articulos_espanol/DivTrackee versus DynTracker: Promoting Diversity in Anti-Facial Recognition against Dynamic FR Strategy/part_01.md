<!-- MinerU 4.0.9, tier advanced, VLM llama.cpp/Vulkan GPU; automatic extraction, tables require comparison with the PDF. -->

[PDF original](../../DivTrackee%20versus%20DynTracker%3A%20Promoting%20Diversity%20in%20Anti-Facial%20Recognition%20against%20Dynamic%20FR%20Strategy.pdf)

# DivTrackee versus DynTracker: Promoting Diversity in Anti-Facial Recognition against Dynamic FR Strategy

Wenshu Fan<sup>∗</sup>

University of Electronic Science and

Technology of China

Chengdu, China

fws@std.uestc.edu.cn

Minxing Zhang

CISPA Helmholtz Center for

Information Security

Saarbrücken, Germany

minxing.zhang@cispa.de

Hongwei Li

University of Electronic Science and

Technology of China

Chengdu, China

hongweili@uestc.edu.cn

Wenbo Jiang†

University of Electronic Science and

Technology of China

Chengdu, China

wenbo\_jiang@uestc.edu.cn

Hanxiao Chen

University of Electronic Science and

Technology of China

Chengdu, China

chenhanxiao.chx@gmail.com

Xiangyu Yue

The Chinese University of Hong Kong

Hong Kong, China

xyyue@cuhk.edu.hk

Michael Backes

CISPA Helmholtz Center for

Information Security

Saarbrücken, Germany

backes@cispa.de

Xiao Zhang†

CISPA Helmholtz Center for

Information Security

Saarbrücken, Germany

xiao.zhang@cispa.de

## Resumen

La adopción generalizada de modelos de facial recognition (FR) plantea serias preocupaciones sobre su posible uso indebido, lo que motiva el desarrollo de anti-facial recognition (AFR) para proteger la privacidad facial del usuario. En este artículo, sostenemos que la static FR strategy, adoptada predominantemente en la literatura previa para evaluar la eficacia de AFR, no puede caracterizar fielmente las capacidades reales de trackers determinados cuyo objetivo es rastrear una identidad objetivo específica. En particular, introducimos DynTracker, una dynamic FR strategy donde la gallery database del modelo se actualiza iterativamente con imágenes recién reconocidas de la identidad objetivo. Sorprendentemente, un enfoque tan simple vuelve ineficaces todas las protecciones de AFR existentes. Para mitigar las amenazas a la privacidad planteadas por DynTracker, abogamos por promover explícitamente la diversidad en las imágenes protegidas por AFR. Hipotetizamos que la falta de diversidad es la causa principal del fracaso de los métodos de AFR existentes. Específicamente, desarrollamos DivTrackee, un método novedoso para generar protecciones de AFR diversas que se basa en un framework de generación de imágenes text-guided y adversarial losses que promueven la diversidad. A través de experimentos exhaustivos en diversos benchmarks de imágenes y feature extractors, demostramos la solidez de DynTracker para romper los métodos de AFR existentes y la superioridad de DivTrackee para evitar que las imágenes faciales de los usuarios sean identificadas por dynamic FR strategies. Creemos que nuestro trabajo puede constituir un paso inicial importante hacia el desarrollo de métodos de AFR más efectivos para proteger la privacidad facial de los usuarios frente a trackers determinados.

## Conceptos CCS

• **Computing methodologies** → **Machine learning**; **Computer vision**; • **Security and privacy** → **Privacy protections**.

## Palabras clave

(Anti-)Facial Recognition; Dynamic Strategy; Privacy Preservation

## Formato de referencia ACM:

Wenshu Fan, Minxing Zhang, Hongwei Li, Wenbo Jiang, Hanxiao Chen, Xiangyu Yue, Michael Backes, and Xiao Zhang. 2025. DivTrackee versus Dyn-Tracker: Promoting Diversity in Anti-Facial Recognition against Dynamic FR Strategy. In Proceedings of the 2025 ACM SIGSAC Conference on Computer and Communications Security (CCS ’25), October 13–17, 2025, Taipei, Taiwan. ACM, New York, NY, USA, 15 pages. [https://doi.org/10.1145/3719027.3744867](https://doi.org/10.1145/3719027.3744867)

## 1 Introducción

Los modelos de facial recognition, donde el objetivo es reconocer la identidad de individuos a partir de imágenes o videos digitales, se han adoptado en diversas aplicaciones del mundo real, tales como el control de seguridad y la gestión de asistencia [3, 24, 26, 37, 38]. Aunque los sistemas de FR mejoran en gran medida la comodidad de nuestra vida cotidiana, su potencial uso indebido genera serias preocupaciones. En particular, tras la promulgación de regulaciones de protección de la privacidad como el GDPR [1] y la CCPA [2], más personas han comenzado a darse cuenta de que los modelos no autorizados de FR pueden amenazar significativamente su privacidad facial. Por consiguiente, ha habido una creciente atención hacia el desarrollo de técnicas de AFR para proteger la privacidad facial de los usuarios que publican sus imágenes en línea [54]. Los métodos existentes pueden dividirse principalmente en dos categorías: gallery-target AFR [9, 10, 21, 47] y query-target AFR [13, 14, 19, 29, 31, 32, 46,

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>∗</sup>Contribución equitativa.</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280">†Autores de correspondencia.</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280">Permission to make digital or hard copies of all or part of this work for personal or classroom use is granted without fee provided that copies are not made or distributed for profit or commercial advantage and that copies bear this notice and the full citation on the first page. Copyrights for components of this work owned by others than the author(s) must be honored. Abstracting with credit is permitted. To copy otherwise, or republish, to post on servers or to redistribute to lists, requires prior specific permission and/or a fee. Request permissions from permissions@acm.org.</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280">CCS ’25, Taipei, Taiwan</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280">© 2025 Copyright held by the owner/author(s). Publication rights licensed to ACM.</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280">ACM ISBN 979-8-4007-1525-9/2025/10</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280">[https://doi.org/10.1145/3719027.3744867](https://doi.org/10.1145/3719027.3744867)</span></small>

51, 56, 58], en función de si las perturbaciones se generan en las imágenes de galería (gallery) o de consulta (query) para evitar el reconocimiento de la identidad objetivo.

Al presenciar la falta de un threat model bien definido en la literatura existente sobre AFR, comenzamos definiendo formalmente el problema como un security game bipartito entre Tracker y Trackee (Sección 3). Utilizamos Tracker para denotar al usuario malicioso que está determinado a rastrear las imágenes en línea de una identidad objetivo específica utilizando FR, mientras que Trackee representa a la identidad objetivo que despliega protecciones de AFR en sus imágenes faciales antes de publicarlas para eludir el rastreo. Al examinar la literatura relevante sobre anti-facial recognition, observamos que todos los métodos de AFR existentes, ya sean gallery-target o query-target, adoptan una static FR strategy en su evaluación.

**Contribución.** Sostenemos que tal suposición estática restringe en gran medida el comportamiento del Tracker al utilizar un modelo de FR, lo cual es insuficiente para caracterizar las capacidades reales de Trackers determinados (Sección 4.1). En particular, proponemos DynTracker, una dynamic FR strategy que actualiza iterativamente la gallery database del modelo de FR del Tracker con las imágenes del Trackee recién identificadas (Sección 4.2). DynTracker enriquece gradualmente la gallery database con más pistas sobre las características faciales del Trackee y el esquema de perturbación de AFR desplegado, permitiéndole descubrir más imágenes faciales del Trackee. Con base en un estudio de caso preliminar, demostramos que DynTracker puede lograr de hecho tasas de éxito de rastreo significativamente más altas que la static FR strategy, volviendo casi completamente ineficaces todos los esquemas de protección de AFR existentes (Sección 4.3).

Asimismo, analizamos las razones subyacentes tras el fallo catastrófico de los métodos de AFR existentes frente a nuestro DynTracker, donde hipotetizamos que la causa principal es la falta de diversidad en las imágenes protegidas por AFR (Sección 5.1). En el caso de AFR basado en modelos generativos (generative-based AFR), en particular, se utiliza típicamente una imagen auxiliar fija de una identidad diferente a la del Trackee para guiar el proceso de generación, lo que conduce a patrones similares compartidos entre las imágenes del Trackee protegidas por AFR. Por lo tanto, una vez que el modelo de FR reconoce unas pocas imágenes protegidas del Trackee y se incluyen en la gallery database, dichas similitudes disminuirán significativamente las tasas de éxito de protección de AFR. Para abordar los riesgos de privacidad planteados por DynTracker, desarrollamos DivTrackee, un método novedoso de generative-based AFR text-guided que se basa en módulos promotores de diversidad y adversarial losses (Sección 5.2). En particular, DivTrackee promueve explícitamente generaciones más diversas de perturbaciones de AFR al seleccionar aleatoriamente una imagen auxiliar de un conjunto de imágenes candidatas y penalizar las similitudes de las imágenes recién generadas con respecto a imágenes protegidas por AFR del Trackee producidas previamente y almacenadas en una cola.

Llevamos a cabo extensos experimentos a través de benchmarks de imágenes faciales y feature extractors para validar la fuerza de DynTracker en la identificación de imágenes del Trackee y la eficacia de protección de DivTrackee frente a dynamic FR strategies (Sección 6). También variamos el conocimiento inicial del Tracker sobre el Trackee y consideramos modelos de facial verification para tener en cuenta posibles variaciones en los comportamientos de rastreo del Tracker. Nuestros resultados confirman que todos los métodos de AFR existentes son altamente vulnerables a dynamic FR strategies. Por ejemplo, un esquema de generative-based AFR state-of-the-art, Clip2Protect [46], logra tasas de éxito de protección inferiores al 20% con respecto a las imágenes del Trackee frente a nuestro DynTracker. En marcado contraste, nuestro DivTrackee reduce significativamente las tasas de éxito de rastreo de DynTracker, logrando consistentemente un incremento de más del 40% en el éxito de la protección mientras preserva una alta calidad visual de las imágenes protegidas por AFR del Trackee. Nuestro trabajo resalta la importancia de adoptar dynamic FR strategies, como nuestro DynTracker, para evaluaciones más rigurosas de los métodos de AFR y también revela la efectividad de promover protecciones de AFR más diversas como contramedidas potenciales. Esperamos que nuestro trabajo pueda servir como un primer paso importante hacia el desarrollo de esquemas de protección de AFR más confiables contra trackers determinados.

En resumen, nuestras contribuciones clave son las siguientes:

• Al caracterizar las limitaciones de las static FR strategies, proponemos una dynamic FR strategy denominada DynTracker, que es fácil de implementar y más potente.
• Para mitigar de forma adaptativa las brechas de privacidad inducidas por DynTracker, proponemos DivTrackee, que genera diversas protecciones de AFR utilizando generación de imágenes text-guided y adversarial losses que promueven la diversidad.
• Experimentos exhaustivos demuestran que DynTracker rompe eficazmente los métodos de AFR existentes en diversos benchmarks de imágenes faciales y feature extractors, y DivTrackee demuestra un rendimiento superior al evitar que las imágenes faciales de los usuarios sean identificadas por dynamic FR strategies.
