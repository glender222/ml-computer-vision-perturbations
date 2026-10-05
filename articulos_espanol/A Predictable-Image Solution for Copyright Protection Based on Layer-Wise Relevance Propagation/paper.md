<!-- MinerU 4.0.9, tier advanced, VLM llama.cpp/Vulkan GPU; automatic extraction, tables require comparison with the PDF. -->

[PDF original](../../A%20Predictable-Image%20Solution%20for%20Copyright%20Protection%20Based%20on%20Layer-Wise%20Relevance%20Propagation.pdf)

Artículo

# Una solución de imagen predecible para copyright protection basada en Layer-Wise Relevance Propagation

**Yougyung Park †, Sieun Kim † e Inwhee Joe \***

Department of Computer Science, Hanyang University, Wangsimni-ro, Seongdong-gu, Seoul 04763, Republic of Korea; eunicepark83@gmail.com (Y.P.); clown14@hanyang.ac.kr (S.K.)

\* Correspondencia: iwjoe@hanyang.ac.kr
† Estos autores contribuyeron por igual a este trabajo.

## Resumen

A medida que los sistemas de inteligencia artificial (IA) se despliegan cada vez más en aplicaciones del mundo real, las preocupaciones relativas al uso no autorizado de imágenes protegidas por derechos de autor durante el entrenamiento de modelos se han vuelto más pronunciadas. En particular, tanto los modelos generativos como los discriminativos pueden internalizar de forma implícita patrones visuales distintivos a partir de datos protegidos, lo que conlleva posibles riesgos éticos y legales incluso tras la eliminación de los datos. En este estudio, proponemos un marco práctico de copyright protection, denominado Predictable-Image Solution (PIS), cuyo objetivo es perturbar el aprendizaje de características visuales protegidas durante el proceso de entrenamiento. PIS aprovecha Layer-Wise Relevance Propagation (LRP) para identificar las regiones de la imagen que contribuyen positivamente a la predicción del modelo y modifica selectivamente dichas regiones utilizando sustitutos visuales no protegidos por derechos de autor, tales como texturas o patrones de imagen benignos. Al enfocarse en regiones semánticamente influyentes en lugar de aplicar perturbaciones globales, el enfoque propuesto interfiere eficazmente con la extracción de características mientras preserva la calidad perceptual y la estructura visual global de la imagen original. Amplios experimentos llevados a cabo en múltiples modelos pre-trained de clasificación de imágenes demuestran que PIS degrada consistentemente el rendimiento de clasificación en las imágenes protegidas, manteniendo al mismo tiempo una alta similitud visual medida mediante métricas perceptuales. Estos resultados indican que PIS ofrece una solución eficaz, model-agnostic y visualmente discreta para mitigar la explotación no autorizada de imágenes protegidas en escenarios prácticos de entrenamiento de IA.

**Palabras clave:** copyright protection; Layer-Wise Relevance Propagation (LRP); explainable AI

![](images/page_0_image_13.jpg)

## Comprobar actualizaciones

Editor académico: Zhengjun Liu

Recibido: 6 de febrero de 2026

Revisado: 5 de marzo de 2026

Aceptado: 14 de marzo de 2026

Publicado: 16 de marzo de 2026

**Copyright:** © 2026 por los autores.

Licenciatario MDPI, Basilea, Suiza.

Este artículo es un artículo de acceso abierto

distribuido bajo los términos y

condiciones de la [licencia Creative Commons](https://creativecommons.org/licenses/by/4.0/)

[Attribution (CC BY).](https://creativecommons.org/licenses/by/4.0/)

## 1. Introducción

La inteligencia artificial (IA) se ha integrado profundamente en diversos aspectos de la vida moderna, incluyendo la atención médica, la comunicación y la generación de imágenes. Entre las diversas modalidades de datos, las imágenes transmiten una información semántica más rica que las descripciones textuales, lo que las hace especialmente valiosas para la comprensión visual y la creación de contenido. Sin embargo, la creación de imágenes de alta calidad exige una creatividad sustancial y un gran esfuerzo humano. Para superar esta limitación, muchos modelos de IA se entrenan utilizando imágenes protegidas por derechos de autor sin el consentimiento de los titulares de dichos derechos, suscitando graves preocupaciones éticas y jurídicas. Dicho uso no autorizado no solo infringe los derechos de los creadores, sino que también socava la motivación artística. El rápido avance de la IA generativa y de los conjuntos de datos a gran escala ha intensificado aún más este problema. Cuando se incluyen imágenes protegidas en los datos de entrenamiento, los modelos de deep learning inevitablemente internalizan las características visuales en sus parámetros. Una vez aprendidas, estas características no pueden eliminarse fácilmente, incluso si los datos originales se eliminan o si el acceso queda restringido [1].

Por consiguiente, los mecanismos de control posteriores al entrenamiento (post-training) son fundamentalmente insuficientes para una copyright protection eficaz.

Las técnicas convencionales de copyright protection, tales como las marcas de agua digitales (watermarking), las perturbaciones invisibles y la autenticación basada en incrustación (embedding-based authentication), adolecen de limitaciones inherentes. Con frecuencia, las marcas de agua pueden eliminarse mediante posprocesamiento, mientras que las perturbaciones invisibles pueden detectarse o neutralizarse mediante algoritmos de restauración avanzados. Además, estos enfoques se aplican típicamente post hoc, lo que los hace estructuralmente incapaces de impedir que los modelos retengan la información aprendida previamente.

Recientemente, la investigación sobre copyright protection y seguridad de modelos se ha centrado cada vez más en técnicas de inteligencia artificial explicable (explainable artificial intelligence, XAI) para analizar el comportamiento del modelo. Métodos como los mapas de saliencia (saliency maps), Grad-CAM y la atribución basada en relevancia han sido ampliamente utilizados para la interpretación de modelos y el análisis de vulnerabilidades. Sin embargo, estas técnicas han permanecido en gran medida descriptivas, y su uso como mecanismos proactivos para proteger los datos de entrenamiento sigue siendo limitado. En contraste, la solución propuesta Predictable-Image Solution (PIS) reutiliza la información de relevancia no meramente como una salida explicativa, sino como una señal de control activo para la copyright protection. En este contexto, una “Predictable-Image” se refiere a una imagen cuya estructura semántica global sigue siendo interpretable para los observadores humanos, al tiempo que está diseñada intencionalmente para inducir una clasificación errónea durante el aprendizaje del modelo. En otras palabras, PIS desacopla explícitamente la predictibilidad perceptual para los humanos de la consistencia predictiva en los modelos de machine learning.

Este enfoque es fundamentalmente distinto de los adversarial examples convencionales, que normalmente introducen perturbaciones imperceptibles en tiempo de inferencia para manipular las salidas del modelo. En su lugar, PIS apunta a la etapa de entrenamiento impidiendo la internalización de características críticas protegidas en los parámetros del modelo. Al aprovechar Layer-Wise Relevance Propagation (LRP) [2,3], el marco propuesto identifica las regiones de la imagen que contribuyen de manera más significativa a las decisiones de clasificación y sustituye selectivamente estas regiones con contenido de reemplazo no protegido. Esta modificación estructurada preserva el contexto semántico global al tiempo que distorsiona sistemáticamente las señales discriminativas que de otro modo se aprenderían. Amplios experimentos a través de múltiples arquitecturas de visión demuestran que PIS degrada eficazmente el rendimiento de clasificación en las imágenes protegidas manteniendo al mismo tiempo una alta similitud visual. Estos resultados indican que PIS constituye un marco de copyright protection práctico, explicable y orientado a la etapa de aprendizaje, en lugar de un mecanismo defensivo post hoc.

El resto de este artículo está organizado de la siguiente manera. La Sección 2 revisa las técnicas existentes de copyright protection y los estudios relacionados basados en XAI, y discute sus limitaciones. La Sección 3 presenta la arquitectura general del PIS propuesto y describe en detalle el método de reconstrucción de imágenes guiado por relevancia. La Sección 4 introduce la configuración experimental y las métricas de evaluación a través de varios modelos. La Sección 5 proporciona un análisis y discusión de los resultados experimentales. Finalmente, se presentan las conclusiones y las direcciones de investigación futuras.

## 2. Trabajos relacionados

## 2.1. Marcas de agua (Watermark)

El watermarking es una de las técnicas más ampliamente adoptadas para la copyright protection en imágenes digitales [4]. En general, las marcas de agua se incrustan en las imágenes como logotipos, textos o patrones visibles o semitransparentes para indicar la autoría y desalentar la reproducción no autorizada. Los enfoques convencionales de watermarking se basan en la incrustación en el dominio espacial o en el dominio de la frecuencia para garantizar la trazabilidad y la verificación de propiedad. Sin embargo, la efectividad del watermarking se ha debilitado significativamente debido a los avances recientes en las tecnologías de edición y restauración de imágenes. Los métodos de inpainting basados en deep learning y los programas comerciales, como Adobe Photoshop, permiten la eliminación de marcas de agua visibles con una degradación visual mínima [5,6]. Como resultado, las marcas de agua a menudo pueden borrarse sin comprometer la calidad de la imagen, lo que genera serias preocupaciones sobre su robustez frente a la reutilización no autorizada. En contraste, la solución propuesta Predictable-Image Solution (PIS) aborda la copyright protection en la etapa de aprendizaje reemplazando las regiones que son más influyentes para la predicción del modelo. Esta estrategia impide que los modelos internalicen las características protegidas por derechos de autor, haciendo que la recuperación de la información visual original sea computacionalmente inviable.

## 2.2. Protección basada en perturbaciones (Perturbation-Based Protection)

Estudios recientes han propuesto técnicas de protección basadas en perturbaciones que insertan modificaciones cuidadosamente diseñadas en las imágenes para perturbar el aprendizaje del modelo o prevenir la extracción de datos [7,8]. Estos enfoques tienen como objetivo prevenir la internalización de características visuales sensibles mediante la inyección de perturbaciones en las imágenes proporcionadas por los usuarios durante el entrenamiento o el fine-tuning. Por ejemplo, Fawkes se centra en proteger la identidad facial contra los sistemas de reconocimiento, mientras que Glaze está diseñado para evitar que los modelos generativos imiten un estilo artístico específico.

Más recientemente, también se han introducido métodos de protección dirigidos a modelos generativos basados en difusión. MetaCloak [9], por ejemplo, emplea una optimización basada en meta-learning para generar perturbaciones que obstaculizan el entrenamiento no autorizado de modelos de difusión. Estos métodos comparten un objetivo similar con PIS en el sentido de que buscan perturbar el aprendizaje no autorizado del modelo. A pesar de su efectividad, la mayoría de los enfoques basados en perturbaciones dependen de una optimización iterativa basada en gradientes y asumen modelos objetivo o pipelines de entrenamiento específicos. En consecuencia, su robustez entre dominios (cross-domain) puede verse limitada, particularmente para imágenes con texturas complejas o en configuraciones de conjuntos de datos a gran escala. Además, el proceso de optimización incurre en un costo computacional sustancial, y la efectividad de la protección puede variar significativamente según la arquitectura objetivo y la configuración del entrenamiento.

En contraste, PIS proporciona un mecanismo de protección determinista y ligero (lightweight) que no requiere optimización iterativa. En lugar de incrustar ruido adversarial, PIS reconstruye las imágenes modificando selectivamente las regiones semánticamente críticas identificadas a través de Layer-Wise Relevance Propagation (LRP). Esta reconstrucción guiada por relevancia preserva la estructura semántica global desde una perspectiva humana, al tiempo que perturba sistemáticamente las pistas discriminativas que de otro modo se internalizarían durante el entrenamiento del modelo. Debido a esta distinción estructural, PIS logra un rendimiento de protección comparativamente estable y reproducible a través de diversos dominios de imágenes y arquitecturas de modelos. Para clarificar aún más las diferencias entre los métodos basados en perturbaciones existentes y PIS, la Tabla 1 resume sus características en términos de modelos objetivo, mecanismos de protección, perceptibilidad, costo computacional y objetivos de protección.

Tabla 1. Comparación de los objetivos específicos de los modelos y la percepción humana.

| Método | Mecanismo | Perceptibilidad humana | Objetivo |
| --- | --- | --- | --- |
| Fawkes | Adversarial perturbation mediante optimización | Imperceptible | Prevenir el reconocimiento de identidad |
| Glaze | Adversarial perturbation con desplazamiento de estilo | Ligeramente perceptible | Prevenir la imitación de estilo |
| MetaCloak | Perturbación en el espacio de características durante el entrenamiento | Principalmente imperceptible | Prevenir el entrenamiento generativo |
| PIS (Ours) | Sustitución de regiones semánticas guiada por LRP | Claramente reconocible por humanos | Protección activa mediante clasificación errónea predecible |

## 2.3. IA explicable (XAI)

La Inteligencia Artificial Explicable (Explainable Artificial Intelligence, XAI) tiene como objetivo interpretar y visualizar los procesos internos de toma de decisiones de las redes neuronales profundas. Entre las técnicas representativas se incluyen Grad-CAM [10], Layer-Wise Relevance Propagation (LRP) [2], LIME [11], SHAP [12] e Integrated Gradients [13]. Estos métodos han sido ampliamente adoptados para mejorar la transparencia, la equidad y la rendición de cuentas en los sistemas de machine learning [12].

Sin embargo, las técnicas existentes de XAI están diseñadas primordialmente con fines diagnósticos. Proporcionan explicaciones post hoc del comportamiento del modelo, pero no previenen intrínsecamente el uso no autorizado de datos protegidos por derechos de autor durante el entrenamiento.

El PIS propuesto amplía la XAI más allá de la interpretabilidad al transformar el análisis de relevancia en un mecanismo activo de copyright protection. Al aprovechar LRP para identificar regiones críticas y modificarlas selectivamente, PIS obstaculiza la extracción de características significativas protegidas. Este enfoque redefine la XAI como una estrategia de defensa proactiva en lugar de una herramienta explicativa pasiva.

## 2.4. Brecha en la investigación (Research Gap)

En resumen, los métodos existentes de copyright protection presentan limitaciones fundamentales. Las técnicas de watermarking dependen de una protección post hoc y pueden eliminarse fácilmente con herramientas modernas de restauración y edición de imágenes. Los enfoques basados en perturbaciones son eficaces para perturbar el entrenamiento del modelo, pero a menudo sufren de inestabilidad entre diferentes dominios de imágenes e incurren en altos costos computacionales. Por su parte, los métodos de inteligencia artificial explicable (XAI) se centran principalmente en interpretar las decisiones del modelo y no impiden directamente el uso no autorizado de imágenes protegidas durante el entrenamiento.

Para abordar esta brecha en la investigación, proponemos un marco novedoso denominado Predictable-Image Solution (PIS), que extiende la explicabilidad hacia un mecanismo de protección activa. Al aprovechar el análisis de relevancia basado en LRP, PIS identifica las regiones que son críticas para la decisión de clasificación del modelo y las sustituye selectivamente para perturbar el proceso de aprendizaje en sí mismo. A diferencia de los enfoques existentes, PIS preserva la calidad perceptual al tiempo que previene proactivamente que los modelos internalicen características protegidas por derechos de autor, ofreciendo una solución práctica e independiente del modelo (model-agnostic) para la copyright protection.

## 3. Metodología

Al entrenar modelos de inteligencia artificial (IA) sobre datos visuales, los píxeles o regiones individuales no contribuyen por igual a la predicción final. Por el contrario, un número limitado de regiones semánticamente sobresalientes domina el proceso de toma de decisiones, mientras que otras desempeñan un papel marginal o incluso supresor. Para identificar y cuantificar explícitamente tales contribuciones, este estudio adopta Layer-Wise Relevance Propagation (LRP) [3], una técnica de explicabilidad que atribuye la salida del modelo de regreso al espacio de entrada.

A partir de la información de relevancia obtenida mediante LRP, proponemos una Predictable-Image Solution (PIS) que modifica selectivamente las regiones de la imagen según su contribución a la clasificación. La idea central consiste en perturbar deliberadamente el aprendizaje de patrones visuales discriminativos alterando las regiones en las que confía el modelo, preservando al mismo tiempo la plausibilidad perceptual general. La Figura 1 ilustra ejemplos de imágenes del conjunto de datos ImageNet-100 procesadas con el enfoque propuesto.

![](images/page_4_image_2.jpg)

<details>
<summary>text_image</summary>

swin_b | pos 50% neg 50%
clean
Image
clean_0
clean_21
clean_42
clean_63
clean_85
clean_106
clean_127
clean_149
prot_0
prot_21
prot_42
prot_63
prot_85
prot_106
prot_127
prot_149
Noise
On
Positive
prot_0
prot_21
prot_42
prot_63
prot_85
prot_106
prot_127
prot_149
Replace
On
Positive
</details>

Figura 1. Ejemplos de imágenes del conjunto de datos ImageNet-100 procesadas utilizando la Predictable-Image Solution (PIS) propuesta. La primera fila muestra las imágenes limpias originales. La segunda fila ilustra la aplicación de ruido en las regiones con contribuciones positivas a la decisión de clasificación, identificadas mediante Layer-Wise Relevance Propagation (LRP). La tercera fila muestra el caso en el que las mismas regiones de relevancia positiva se reemplazan con imágenes sustitutas libres de derechos de autor. En todas las imágenes protegidas, las regiones con contribuciones negativas se modifican consistentemente mediante una transformación de color basada en desenfoque (blur) a fin de preservar la estructura visual global mientras se perturban selectivamente las características discriminativas utilizadas por el clasificador.

## 3.1. Propagación de relevancia (Relevance Propagation)

Layer-Wise Relevance Propagation (LRP) es un método de atribución diseñado para explicar la predicción de un modelo redistribuyendo la puntuación de salida hacia atrás a través de las capas de la red conservando la relevancia total [14]. Comenzando desde la neurona de salida correspondiente a la clase predicha, las puntuaciones de relevancia se propagan capa por capa hacia la entrada, dando como resultado un mapa de relevancia que refleja la contribución de cada unidad de entrada. Formalmente, la regla de propagación de relevancia para una neurona de entrada $j$ viene dada por:

$$
R _ {j} = \sum_ {k} \frac {a _ {j} w _ {j k}}{\epsilon + \sum_ {0 , j} a _ {j} w _ {j k}} R _ {k}, \tag {1}
$$

donde $R _ { j }$ denota la puntuación de relevancia de la neurona $j$, $a _ { j }$ es su activación, $w _ { j k }$ representa el peso que conecta las neuronas $j$ y $k$, y $\epsilon$ es una pequeña constante estabilizadora introducida para evitar inestabilidad numérica. En este trabajo, seguimos la formulación estándar comúnmente adoptada en la literatura de Layer-Wise Relevance Propagation (LRP) [2,15]. La función de $\epsilon$ es garantizar una propagación de relevancia estable y no actuar como un hiperparámetro ajustable. Debido a su propiedad de conservación de relevancia, LRP proporciona una localización fiel de las regiones críticas para la decisión y permite el análisis directo de cómo se desplazan las dependencias semánticas bajo los mecanismos de protección de imágenes.

## 3.2. Reemplazo selectivo de regiones (Selective Region Replacement)

Utilizando LRP, se calculan las puntuaciones de relevancia para cada imagen de entrada y posteriormente se particionan en regiones de contribución positiva y negativa [15]. Las regiones con relevancia positiva corresponden a características que respaldan activamente la clase predicha, mientras que las regiones con relevancia negativa suprimen o contrarrestan la predicción. Sobre la base de esta distinción, el PIS propuesto aplica modificaciones dirigidas a cada tipo de región. El procedimiento general de protección de imágenes basado en relevancia se resume en el Algoritmo 1.

Algoritmo 1 Protección de imágenes basada en relevancia mediante LRP

<div class="docvortex-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Require: Image $I$, relevance ratios $(\alpha, \beta)$
Ensure: Protected image $I'$
$R \leftarrow \text{LRP}(I)$
$R^{+} \leftarrow \text{Top-}\alpha\%$ of $R$
$R^{-} \leftarrow \text{Bottom-}\beta\%$ of $R$
Replace $R^{+}$ with external image patches
Visual Processing of $R^{-}$
Output $I'$
</div>

Específicamente, las regiones con relevancia positiva se reemplazan con contenido visual libre de derechos de autor, como patrones de textura, ruido aleatorio o imágenes de objetos semánticamente no relacionados. Esta estrategia de reemplazo se elige deliberadamente por encima de la simple inyección de ruido, dado que los modelos modernos de deep learning pueden ignorar o suprimir el ruido aleatorio a través de interpolación o mecanismos internos de eliminación de ruido (denoising). Por el contrario, los patrones visualmente complejos originados a partir de diferentes fuentes de imágenes introducen ambigüedad durante el entrenamiento y la inferencia, impidiendo así que el modelo aprenda de manera consistente las características discriminativas originales. Las regiones con relevancia negativa reciben un tratamiento diferente. Dado que estas regiones no contribuyen directamente a la clasificación correcta, una modificación agresiva resulta innecesaria. En su lugar, se aplican transformaciones suaves tales como desenfoque (blurring), efecto mosaico o ajuste de color. En este estudio, se adopta el desenfoque para mantener la coherencia visual general al tiempo que se introduce interferencia perceptual adicional.

## 3.3. Integración en Predictable-Image Solution (PIS)

Los procedimientos de reemplazo selectivo y ajuste visual se integran como un componente central de la solución propuesta Predictable-Image Solution (PIS). A diferencia de las estrategias de protección post hoc que intentan mitigar los riesgos de privacidad o de derechos de autor después del entrenamiento del modelo, PIS está explícitamente diseñado para perturbar el aprendizaje en la etapa de entrenamiento. Al impedir la internalización de representaciones visuales críticas asociadas con el contenido protegido por derechos de autor, el enfoque propuesto busca reducir fundamentalmente la persistencia de las características protegidas, incluso en escenarios donde los conjuntos de datos se eliminan o modifican con posterioridad. La Figura 1 presenta ejemplos cualitativos de imágenes procesadas mediante PIS, donde las operaciones de inyección de ruido y de reemplazo se aplican a regiones con relevancia positiva y negativa en una proporción del 50%. Para analizar más a fondo el impacto en el comportamiento del modelo, la Figura 2 presenta las matrices de confusión obtenidas tras entrenar un modelo representativo ViT-B/16 bajo diferentes configuraciones de protección.

![](images/page_5_chart_7.jpg)

<details>
<summary>heatmap</summary>

| True label \ Predicted label | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 1.0 | — | — | — | — | — | — | — | — | — | — |
| 1 | — | 1.0 | — | — | — | — | — | — | — | — | — |
| 2 | — | — | 1.0 | — | — | — | — | — | — | — | — |
| 3 | — | — | — | 1.0 | — | — | — | — | — | — | — |
| 4 | — | — | — | — | 1.0 | — | — | — | — | — | — |
| 5 | — | — | — | — | — | 1.0 | — | — | — | — | — |
| 6 | — | — | — | — | — | — | 1.0 | — | — | — | — |
| 7 | — | — | — | — | — | — | — | 1.0 | — | — | — |
| 8 | — | — | — | — | — | — | — | — | 1.0 | — | — |
| 9 | — | — | — | — | — | — | — | — | — | 1.0 | — |
</details>

(a)

![](images/page_5_chart_9.jpg)

<details>
<summary>heatmap</summary>

| True label | Predicted label 0 | Predicted label 1 | Predicted label 2 | Predicted label 3 | Predicted label 4 | Predicted label 5 | Predicted label 6 | Predicted label 7 | Predicted label 8 | Predicted label 9 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | ~0.15 | ~0.55 | ~0.25 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.35 | ~0.35 |
| 1 | ~0.15 | ~0.75 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.35 |
| 2 | ~0.15 | ~0.45 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.65 | ~0.65 | ~0.65 |
| 3 | ~0.15 | ~0.55 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.45 | ~0.45 | ~0.45 |
| 4 | ~0.15 | ~0.35 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.75 | ~0.75 | ~0.75 |
| 5 | ~0.15 | ~0.45 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.65 | ~0.65 | ~0.65 |
| 6 | ~0.15 | ~0.45 | ~0.25 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.35 | ~0.35 | ~0.35 |
| 7 | ~0.15 | ~0.75 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.35 | ~0.35 | ~0.35 |
| 8 | ~0.15 | ~0.45 | ~0.45 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.35 | ~0.35 | ~0.35 |
| 9 | ~0.15 | ~0.65 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.15 | ~0.45 | ~0.45 | ~0.45 |
</details>

(b)

![](images/page_5_chart_11.jpg)

<details>
<summary>heatmap</summary>

| True label | Predicted label 0 | Predicted label 1 | Predicted label 2 | Predicted label 3 | Predicted label 4 | Predicted label 5 | Predicted label 6 | Predicted label 7 | Predicted label 8 | Predicted label 9 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | ~0.65 | — | — | — | ~0.35 | — | — | — | — | — |
| 1 | ~0.35 | — | — | — | ~0.65 | — | ~0.25 | — | — | — |
| 2 | ~0.15 | — | ~0.15 | — | ~0.70 | — | ~0.45 | ~0.35 | — | — |
| 3 | — | — | — | — | ~0.70 | — | ~0.45 | — | — | — |
| 4 | ~0.15 | — | — | — | ~0.75 | — | ~0.25 | — | — | — |
| 5 | ~0.20 | — | — | — | ~0.45 | — | ~0.55 | — | — | — |
| 6 | ~0.35 | — | — | — | ~0.45 | — | ~0.30 | — | — | — |
| 7 | ~0.15 | — | — | — | ~0.65 | — | ~0.25 | ~0.20 | ~0.15 | — |
| 8 | ~0.25 | — | — | — | ~0.45 | — | ~0.25 | ~0.15 | — | — |
| 9 | ~0.35 | — | — | — | ~0.25 | — | ~0.45 | — | — | — |
</details>

(c)

Figura 2. Matrices de confusión obtenidas al entrenar el modelo representativo ViT-B/16 utilizando imágenes con un 50% de protección PIS aplicada a regiones positivas y negativas bajo configuraciones (a) limpias, (b) basadas en ruido y (c) basadas en reemplazo. Mientras que las imágenes limpias preservan predicciones de clase equilibradas, las imágenes protegidas basadas en ruido y reemplazo exhiben predicciones sesgadas hacia clases específicas en lugar de las correctas.

## 3.4. Estrategia de selección de imágenes de reemplazo (Replacement Image Selection Strategy)

En la perturbación basada en reemplazo, las imágenes sustitutas se recopilaron a partir de imágenes públicamente disponibles en Wikimedia Commons [16]. Estas imágenes eran externas al conjunto de datos de evaluación original, minimizando así una posible superposición en la distribución. No se impusieron criterios específicos ni restricciones a nivel de clase durante la selección.

Para examinar si la degradación de rendimiento observada depende del contenido de reemplazo específico, comparamos las perturbaciones de ruido, textura y reemplazo bajo condiciones experimentales idénticas. Como se muestra en la Tabla 2, los tres tipos de perturbación arrojaron un accuracy comparable (Accuracy ≈ 0.09), lo que indica un efecto de degradación consistente a través de los diferentes tipos de perturbación. Esto sugiere que la degradación no depende fuertemente de las características semánticas de las imágenes de reemplazo.

Tabla 2. Resultados experimentales cuantitativos de las tres variantes de protección PIS (ruido, reemplazo y textura) en los conjuntos de datos ImageNet-100, CIFAR-10 y CIFAR-100 (subconjunto). Se reportan el accuracy de clasificación, el promedio de confianza de predicción (mean confidence) y la entropía media para evaluar el grado de perturbación del aprendizaje, mientras que se miden el SSIM y el PSNR para evaluar la preservación de la calidad perceptual de la imagen. Debido a que las imágenes de CIFAR-10 tienen una resolución muy baja (32 × 32), las métricas PSNR y SSIM son menos confiables y proporcionan información limitada sobre la calidad perceptual.

<table><tr><td>Dataset</td><td>Method</td><td>Accuracy</td><td>Mean_Confidence</td><td>Mean_Entropy</td><td>Mean_SSIM</td><td>Mean_PSNR</td></tr><tr><td rowspan="3">ImageNet-100</td><td>noise</td><td>0.090909</td><td>0.918324</td><td>0.341966</td><td>0.254969</td><td>12.78203</td></tr><tr><td>replace</td><td>0.090909</td><td>0.515685</td><td>1.889622</td><td>0.296057</td><td>14.1461</td></tr><tr><td>texture</td><td>0.090909</td><td>0.606044</td><td>1.52149</td><td>0.290198</td><td>15.12889</td></tr><tr><td rowspan="3">CIFAR-10</td><td>noise</td><td>0.0</td><td>0.912624</td><td>5.049304</td><td>-</td><td>-</td></tr><tr><td>replace</td><td>0.0</td><td>0.903358</td><td>4.930081</td><td>-</td><td>-</td></tr><tr><td>texture</td><td>0.0</td><td>0.907134</td><td>4.866816</td><td>-</td><td>-</td></tr><tr><td rowspan="3">CIFAR-100 (Subset)</td><td>noise</td><td>0.0</td><td>0.305385</td><td>1.904548</td><td>0.167871</td><td>12.5146</td></tr><tr><td>replace</td><td>0.0</td><td>0.290138</td><td>1.932238</td><td>0.257229</td><td>15.22556</td></tr><tr><td>texture</td><td>0.0</td><td>0.306736</td><td>1.910972</td><td>0.291306</td><td>16.49597</td></tr></table>

Además, la perturbación por reemplazo produjo una entropía predictiva media mayor en comparación con la perturbación basada en ruido, lo que indica una mayor incertidumbre en la salida en lugar de un sesgo hacia una clase específica. Asimismo, las métricas de similitud de imagen muestran que las perturbaciones de reemplazo y de textura dieron lugar a valores de SSIM y PSNR ligeramente superiores en comparación con la perturbación por ruido. A pesar de estas diferencias en la similitud estructural y en la fidelidad de la señal, el accuracy de clasificación permaneció igualmente degradado. Esta observación respalda aún más la interpretación de que la caída en el rendimiento no es atribuible únicamente a una distorsión de la imagen a bajo nivel, sino que refleja la sensibilidad del modelo a la sustitución a nivel de contenido. Por lo tanto, la degradación de rendimiento observada se interpreta como el reflejo de una vulnerabilidad general al reemplazo de contenido, en lugar de artefactos de una selección de imágenes particular o una simple corrupción a nivel de píxel.

## 4. Experimentos

Esta sección describe la configuración experimental y el protocolo de evaluación utilizados para verificar la efectividad del marco de copyright protection propuesto, denominado Predictable-Image Solution (PIS). El objetivo de los experimentos es analizar cómo las imágenes protegidas con PIS afectan el rendimiento de aprendizaje e inferencia de los modelos de clasificación de imágenes, evaluando cuantitativamente de manera simultánea si la calidad perceptual y las estructuras semánticas reconocibles por el ser humano se preservan tras la aplicación del mecanismo de protección.

Los datos experimentales constan de imágenes limpias (originales) y sus correspondientes imágenes protegidas generadas mediante PIS. Para examinar explícitamente el impacto del mecanismo de protección en el aprendizaje del modelo, todos los experimentos inicializan los modelos con pesos pre-trained y llevan a cabo un fine-tuning adicional exclusivamente sobre las imágenes protegidas durante un número limitado de epochs. Este diseño permite observar directamente cómo influyen las imágenes protegidas en la dinámica de aprendizaje.

Las imágenes protegidas mediante PIS se generan a partir de mapas de relevancia obtenidos a través de Layer-Wise Relevance Propagation (LRP). De acuerdo con la predicción del modelo, las regiones de la imagen se dividen en regiones de relevancia positiva y regiones de relevancia negativa. Las regiones de relevancia positiva, que contribuyen con mayor fuerza a las decisiones de clasificación, se modifican mediante inyección de ruido, reemplazo con imágenes sustitutas libres de derechos de autor o sustitución de textura. Por el contrario, las regiones de relevancia negativa se someten únicamente a transformaciones de color limitadas, preservando así la estructura global y la coherencia semántica de la imagen al tiempo que se perturban selectivamente las características visuales discriminativas.

El rendimiento del modelo se evalúa comparando el accuracy de clasificación antes y después de la protección. En concreto, el accuracy obtenido al entrenar y evaluar los modelos en imágenes limpias (denotado como acc\_clean) se compara con el accuracy alcanzado tras un entrenamiento adicional sobre imágenes protegidas (denotado como acc\_protected). Esta comparación proporciona una medida cuantitativa del colapso del aprendizaje (learning collapse) inducido por PIS. La calidad perceptual de las imágenes protegidas se evalúa adicionalmente mediante los valores promedio de Peak Signal-to-Noise Ratio (PSNR) y Structural Similarity Index (SSIM). Además, se calcula la cosine similarity entre las imágenes limpias y protegidas para analizar las alteraciones a nivel de características ocasionadas por el proceso de protección.

La evaluación abarca un conjunto diverso de arquitecturas pre-trained, incluyendo MobileNet, ResNet-50, EfficientNet, ViT-B/16 [17], Swin-B y DeiT-B/16, cubriendo tanto redes neuronales convolucionales (CNN) como modelos basados en Transformer. Esta amplia configuración experimental demuestra que el marco PIS propuesto opera de manera agnóstica al modelo (model-agnostic).

Los experimentos se llevan a cabo sobre conjuntos de datos públicamente disponibles con diferentes resoluciones, complejidades visuales y escalas de datos. ImageNet-100 [18,19] es un subconjunto del conjunto de datos a gran escala ImageNet que consta de 100 clases seleccionadas aleatoriamente e imágenes naturales de alta resolución. CIFAR-10 [20] es un conjunto de datos de 10 clases compuesto por imágenes en color de 32 × 32 y se utiliza para analizar los efectos del colapso del aprendizaje en entornos de baja resolución. Además, se construye un subconjunto a pequeña escala de CIFAR-100 seleccionando 10 clases y muestreando 15 imágenes por clase, lo que da un total de 150 imágenes.

Todos los modelos se entrenan bajo configuraciones de optimización idénticas. Se emplea cross-entropy loss como función objetivo y se utiliza el optimizador Adam para el entrenamiento. La tasa de aprendizaje (learning rate) inicial se fija en $1 \times 1 0 ^ { - 3 }$ y el entrenamiento se limita intencionalmente a tres epochs para reflejar el régimen de datos a pequeña escala y observar directamente si las imágenes protegidas pueden inducir el colapso del aprendizaje incluso bajo esquemas cortos de fine-tuning. No se aplica ningún programa de ajuste de la tasa de aprendizaje (learning rate scheduling).

Al limitar el número de imágenes por clase en todos los conjuntos de datos, el diseño experimental permite un examen preciso del colapso del aprendizaje y de la degradación en el rendimiento discriminativo causada por el mecanismo de protección. Esta configuración facilita el análisis de la estabilidad y la generalizabilidad de PIS a través de conjuntos de datos con diferentes resoluciones y escalas de datos.

## 5. Resultados de las pruebas (Test Results)

Esta sección presenta los resultados experimentales obtenidos utilizando la solución propuesta Predictable-Image Solution (PIS) en el conjunto de datos ImageNet-100. El análisis se centra en cómo se ve afectado el rendimiento de clasificación cuando se utilizan imágenes protegidas con PIS como entradas, examinando simultáneamente si la calidad visual y la similitud en el espacio de características se preservan o se perturban.

## 5.1. Impacto de PIS en el accuracy de clasificación

La Figura 3 ilustra la variación en el accuracy de clasificación (acc\_protected) en función de la fuerza de protección, controlada por las proporciones de regiones con relevancia positiva y negativa. La Figura 3a muestra los resultados obtenidos utilizando PIS basado en ruido, mientras que la Figura 3b presenta los resultados para PIS basado en reemplazo. El esquema de protección basado en ruido degrada significativamente el rendimiento de clasificación al perturbar las regiones críticas para la relevancia. En particular, para las arquitecturas basadas en Transformer, se observa una recuperación parcial del accuracy a medida que aumenta la proporción de regiones con relevancia positiva. Este comportamiento sugiere que estos modelos pueden compensar parcialmente las perturbaciones inducidas por el ruido cuando aún se dispone de suficiente información discriminativa. Por el contrario, el esquema de protección basado en reemplazo sustituye selectivamente las pistas semánticas de decisión mientras preserva de forma relativa la similitud en el espacio de características. En consecuencia, el accuracy de clasificación se degrada de manera más gradual, pero se mantiene consistentemente bajo en todas las proporciones de protección, demostrando un efecto de supresión más estable y controlado en comparación con la protección basada en ruido.

![](images/page_8_image_5.jpg)

Figura 3. Accuracy de clasificación en función de la proporción de protección aplicada por la solución propuesta Predictable-Image Solution (PIS). La proporción de protección controla el porcentaje de regiones con relevancia positiva y negativa modificadas según el mapa de relevancia obtenido mediante Layer-Wise Relevance Propagation (LRP). (a) presenta los resultados cuando se inyecta ruido en las regiones que contribuyen positivamente, mientras que (b) muestra los resultados cuando dichas regiones se reemplazan con contenido sustituto libre de derechos de autor. Los resultados ilustran cómo el incremento en la fuerza de protección influye en la fiabilidad de predicción del modelo bajo diferentes estrategias de protección.

## 5.2. Evaluación de la calidad visual

La Figura 4 presenta una evaluación cuantitativa de la calidad visual de las imágenes protegidas con PIS mediante las métricas PSNR y SSIM. El eje x denota la proporción de regiones de relevancia positiva reemplazadas con ruido o con imágenes sustitutas. Es de destacar que las curvas correspondientes a todos los modelos evaluados prácticamente se superponen en las diferentes proporciones de protección. Esta observación indica que el método de protección propuesto mantiene una calidad visual consistente independientemente de la arquitectura del modelo subyacente. Estos resultados confirman que PIS introduce una degradación perceptual mínima al tiempo que logra una perturbación eficaz del aprendizaje del modelo.

![](images/page_9_image_2.jpg)

Figura 4. Evaluación cuantitativa de la calidad visual respecto a la proporción de protección. (a,b) muestran las variaciones de PSNR tras aplicar PIS basado en ruido y en reemplazo, respectivamente, mientras que (c,d) presentan los resultados correspondientes de SSIM bajo las mismas condiciones.

## 5.3. Análisis de similitud en el espacio de características

La relación entre la similitud en el espacio de características y el accuracy de clasificación después de aplicar PIS se muestra en la Figura 5. A pesar de las variaciones en las proporciones de protección, se mantiene un régimen de accuracy consistentemente bajo, lo que indica que PIS no es sensible a configuraciones hiperparamétricas específicas y puede suprimir de manera confiable el rendimiento de clasificación en una amplia gama de configuraciones. Se observa una excepción en el modelo Swin-B, que exhibe un accuracy relativamente más alto alrededor de un valor de cosine similarity de aproximadamente 0.6 en comparación con otras arquitecturas. Adicionalmente, bajo el esquema de protección basado en ruido, ocurre un rebote limitado del accuracy en el rango de cosine similarity de aproximadamente 0.85–0.9. En contraste, el esquema de protección basado en reemplazo suprime eficazmente el rendimiento de clasificación en todos los modelos manteniendo una cosine similarity comparativamente más alta, lo que demuestra un comportamiento de perturbación de características semánticas más estable [21].

![](images/page_9_chart_6.jpg)

<details>
<summary>line</summary>

| Cosine Similarity (Flattened Features) | deit_b16 | efficientnet_b0 | mobilenet_v2 | resnet50 | swin_b | vit_b16 |
| --- | --- | --- | --- | --- | --- | --- |
| 0.60 | — | ~0.19 | — | 0.10 | ~0.49 | ~0.23 |
| 0.70 | — | ~0.16 | — | 0.10 | ~0.18 | ~0.15 |
| 0.80 | — | ~0.12 | ~0.11 | ~0.11 | ~0.13 | ~0.10 |
| 0.85 | ~0.07 | — | ~0.13 | ~0.10 | ~0.16 | ~0.10 |
| 0.90 | — | ~0.09 | ~0.12 | ~0.09 | ~0.22 | ~0.08 |
</details>

(a)

![](images/page_9_chart_8.jpg)

<details>
<summary>line</summary>

| Cosine Similarity (Flattened Features) | deit_b16 | efficientnet_b0 | mobilenet_v2 | resnet50 | swin_b | vit_b16 |
| --- | --- | --- | --- | --- | --- | --- |
| 0.77 | ~0.17 | ~0.14 | ~0.12 | 0.10 | 0.52 | ~0.16 |
| 0.83 | ~0.13 | ~0.16 | ~0.13 | 0.10 | 0.25 | ~0.15 |
| 0.88 | ~0.11 | ~0.12 | ~0.16 | 0.10 | 0.21 | ~0.11 |
| 0.90 | ~0.07 | ~0.10 | ~0.15 | 0.10 | 0.22 | ~0.10 |
</details>

(b)

Figura 5. Relación entre la cosine similarity de las representaciones de características y el accuracy de clasificación después de aplicar PIS. (a) corresponde a PIS basado en ruido y (b) corresponde a PIS basado en reemplazo.

## 5.4. Prueba de robustez (Robustness Test)

Evaluamos la robustez del modelo bajo compresión JPEG (jpeg70), recorte aleatorio (crop90) y desenfoque gaussiano (Gaussian blur, σ = 1). La Figura 6 presenta el accuracy robusto (robust accuracy) de cada modelo bajo estas transformaciones.

![](images/page_10_chart_4.jpg)

<details>
<summary>bar</summary>

| model | blur | crop 90 | jpeg 70 |
| --- | --- | --- | --- |
| efficientnet_b0 | ~0.13 | ~0.14 | ~0.09 |
| mobilenet_v2 | ~0.22 | ~0.23 | ~0.22 |
| resnet50 | ~0.15 | ~0.17 | ~0.14 |
| swin_b | ~0.37 | ~0.64 | ~0.51 |
| deit_b16 | ~0.29 | ~0.47 | ~0.47 |
| vit_b16 | ~0.32 | ~0.52 | ~0.52 |
</details>

Figura 6. Accuracy robusto de varios modelos pre-trained bajo transformaciones comunes de imagen, incluyendo compresión JPEG (calidad 70), recorte (crop 90) y desenfoque (blur). Esta figura compara visualmente cómo varía el rendimiento de clasificación en imágenes protegidas con PIS entre diferentes arquitecturas de modelos, destacando los efectos dependientes del modelo del mecanismo de protección.

Los resultados experimentales muestran que los modelos basados en Transformer (Swin-B, ViT-B16 y DeiT-B16) alcanzaron generalmente un accuracy robusto más alto que los modelos basados en CNN (ResNet50, EfficientNet-B0 y MobileNet-V2). En particular, Swin-B alcanzó la mayor robustez frente a transformaciones espaciales, registrando un accuracy robusto de 0.633 bajo crop90. ViT-B16 también mantuvo un rendimiento estable bajo jpeg70 y crop90, alcanzando valores de accuracy robusto de 0.52 y 0.51, respectivamente.

En contraste, ResNet50 exhibió un accuracy robusto relativamente bajo (0.13–0.16). Sin embargo, como se muestra en la Tabla 3, la cosine similarity se mantuvo comparativamente alta en todas las transformaciones. Esta observación sugiere que, aunque las representaciones de características permanecen relativamente estables tras la transformación, la frontera de decisión final aún puede ser sensible a perturbaciones en la entrada [22]. En otras palabras, la similitud a nivel de representación no garantiza necesariamente la robustez en la clasificación [23].

Tabla 3. Robustez del modelo ResNet-50 bajo transformaciones comunes de imagen, incluyendo compresión JPEG, recorte y desenfoque. Para cada transformación se reportan tanto el accuracy de clasificación (robust accuracy) como la cosine similarity entre las imágenes limpias y protegidas. Esta tabla demuestra que las imágenes protegidas con PIS retienen ciertas características de representación mientras inducen una degradación controlada en el rendimiento del modelo.

| Transform | Robust_ACC | Cosine_Similarity |
| --- | --- | --- |
| jpeg 70 | 0.13 | 0.74 |
| crop 90 | 0.16 | 0.79 |
| blur | 0.14 | 0.77 |

Estos hallazgos indican que las arquitecturas basadas en self-attention global pueden exhibir una robustez comparativamente mayor a la compresión y a las transformaciones espaciales que las arquitecturas CNN que dependen principalmente de campos receptivos locales.

La Figura 7 compara el accuracy protegido (protected accuracy) bajo condiciones de compresión JPEG y recorte a través de diferentes proporciones de protección (replace\_pos75\_neg25, replace\_pos50\_neg50 y replace\_pos25\_neg75).

![](images/page_11_chart_4.jpg)

<details>
<summary>bar</summary>

| model | protected (no transform) | jpeg 70 | jpeg 50 | crop 0.8 | crop 0.7 |
| --- | --- | --- | --- | --- | --- |
| resnet50 | ~0.075 | ~0.04 | ~0.10 | ~0.14 | ~0.065 |
| efficientnet_b0 | ~0.12 | ~0.10 | ~0.06 | ~0.075 | ~0.115 |
| mobilenet_v2 | ~0.125 | ~0.045 | ~0.045 | ~0.105 | ~0.075 |
| swin_b | ~0.115 | ~0.205 | ~0.245 | ~0.08 | ~0.04 |
| deit_b16 | ~0.095 | ~0.195 | ~0.185 | ~0.235 | ~0.095 |
| vit_b16 | ~0.08 | ~0.10 | ~0.045 | ~0.065 | ~0.04 |
</details>

Figura 7. Accuracy protegido de imágenes protegidas con PIS bajo transformaciones extendidas, incluyendo compresión JPEG (calidad 70, 50) y recorte (crop 0.8, 0.7). Esta figura permite evaluar qué tan bien mantienen las imágenes protegidas el rendimiento de clasificación bajo condiciones alteradas y cómo varía el grado de colapso del aprendizaje a través de diferentes escenarios de transformación.

En todos los modelos, el clean accuracy se mantuvo alto (0.85–1.0), mientras que el protected accuracy se mantuvo consistentemente dentro de un rango bajo (0.06–0.13). Esto demuestra que el mecanismo de protección propuesto preserva el rendimiento de clasificación en los datos limpios mientras reduce intencionalmente el accuracy para las muestras protegidas.

Bajo transformaciones JPEG, Swin-B y DeiT-B16 exhibieron un protected accuracy relativamente mayor en comparación con los modelos basados en CNN. Esta tendencia puede atribuirse a la capacidad de las arquitecturas Transformer para integrar información contextual más amplia a través de self-attention global, lo que las hace relativamente menos sensibles a distorsiones locales de alta frecuencia introducidas por la compresión JPEG. Incluso cuando los detalles de grano fino se degradan parcialmente, la información estructural global puede seguir preservándose, permitiendo que la señal de protección permanezca parcialmente efectiva.

Además, DeiT-B16 mostró un protected accuracy comparativamente más alto bajo la condición crop0.8. Al ser un modelo basado en Vision Transformer, DeiT-B16 divide las imágenes en tokens como parches (patches) y modela las relaciones globales a través de self-attention [24]. Cuando se preserva la región central, como en crop0.8, la información contextual global aún puede utilizarse eficazmente [17]. Este comportamiento es consistente con hallazgos previos que indican que los mecanismos de self-attention capturan dependencias globales más allá de los campos receptivos locales [25]. Por el contrario, los modelos basados en CNN pueden experimentar una mayor degradación de las señales de protección cuando las regiones espaciales se eliminan parcialmente.

Es importante destacar que, incluso bajo transformaciones relativamente fuertes tales como JPEG50 y crop0.8, el protected accuracy se mantuvo dentro del rango de 0.04–0.24. Este resultado indica que el método propuesto retiene cierto grado de persistencia de protección frente a manipulaciones comunes de imágenes.

En general, el mecanismo de protección propuesto demuestra un rendimiento estable bajo escenarios realistas de transformación de imágenes.

## 5.5. Análisis de la matriz de confusión (Confusion Matrix Analysis)

La Figura 2 revela un modo crítico de fallo inducido por el PIS propuesto que se extiende más allá de una simple reducción en el accuracy de clasificación. Para imágenes limpias, las clasificaciones erróneas se concentran mayormente entre clases semánticamente relacionadas, formando patrones de confusión estructurados. En contraste, las predicciones para imágenes protegidas basadas en ruido y en reemplazo pierden esta organización semántica, colapsando hacia un pequeño subconjunto de clases o distribuyéndose de manera irregular. Este comportamiento indica que la degradación observada en el rendimiento no es causada meramente por una mayor incertidumbre, sino que refleja un colapso de la comprensión semántica. Al preservar la apariencia global de la imagen mientras perturba las regiones críticas de relevancia, PIS impide que los modelos exploten características discriminativas de clase, dando lugar a predicciones que ya no se alinean con las relaciones semánticas entre clases.

## 5.6. Resumen cuantitativo (Quantitative Summary)

La Tabla 4 resume las métricas de evaluación cuantitativa, incluyendo el accuracy de clasificación en imágenes limpias (acc\_clean), el accuracy de clasificación tras el reentrenamiento en imágenes protegidas con PIS (acc\_protected), los valores promedio de PSNR y SSIM, y el promedio de la cosine similarity entre vectores de características aplanados (flattened feature vectors). Todos los modelos evaluados logran un accuracy de clasificación casi perfecto en imágenes limpias. Sin embargo, cuando se utilizan imágenes protegidas como entradas, se observa consistentemente una degradación sustancial del rendimiento. Por ejemplo, el modelo DeiT-B/16 mantiene un accuracy de 1.0 en imágenes limpias, mientras que su accuracy promedio cae a 0.116 al ser evaluado en imágenes protegidas con PIS. Es importante destacar que esta degradación ocurre incluso cuando la calidad visual no se destruye por completo. Para el modelo DeiT-B/16, los valores promedio de PSNR y SSIM de las imágenes protegidas basadas en ruido son de 9.95 y 0.21, respectivamente, mientras que la protección basada en reemplazo produce valores más altos de 12.32 y 0.26 bajo proporciones de protección distintas de cero. Estos resultados indican que las imágenes protegidas siguen siendo perceptualmente reconocibles para los observadores humanos mientras se convierten en entradas no confiables para los modelos de deep learning. En general, los resultados experimentales demuestran que el marco PIS propuesto perturba eficazmente el rendimiento de clasificación basado en deep learning minimizando al mismo tiempo la degradación perceptual. Esto confirma la viabilidad práctica de PIS como un mecanismo de copyright protection que preserva la interpretabilidad humana al tiempo que previene la explotación no autorizada del modelo.

Tabla 4. Resumen de resultados de PIS para cada modelo. Los parámetros α y β denotan las proporciones de aplicación de ruido y reemplazo, respectivamente.

<table><tr><td>Model</td><td>Solution</td><td>α</td><td>β</td><td>Acc_Clean</td><td>Acc_Protected</td><td>Psnr_Mean</td><td>Ssim_Mean</td><td>Cosine_Flat_Mean</td></tr><tr><td rowspan="10">DeiT-B/16</td><td rowspan="5">noise</td><td>0</td><td>100</td><td>1.0</td><td>0.086667</td><td>14.69491</td><td>0.31905</td><td>0.892694</td></tr><tr><td>10</td><td>90</td><td>1.0</td><td>0.073333</td><td>12.81076</td><td>0.298929</td><td>0.856513</td></tr><tr><td>25</td><td>75</td><td>1.0</td><td>0.113333</td><td>10.95512</td><td>0.262094</td><td>0.799997</td></tr><tr><td>50</td><td>50</td><td>1.0</td><td>0.16</td><td>8.781365</td><td>0.190377</td><td>0.698234</td></tr><tr><td>75</td><td>25</td><td>1.0</td><td>0.186667</td><td>7.265331</td><td>0.106383</td><td>0.594237</td></tr><tr><td rowspan="5">replace</td><td>0</td><td>100</td><td>1.0</td><td>0.066667</td><td>14.83456</td><td>0.322423</td><td>0.902163</td></tr><tr><td>10</td><td>90</td><td>1.0</td><td>0.066667</td><td>14.20433</td><td>0.316984</td><td>0.896518</td></tr><tr><td>25</td><td>75</td><td>1.0</td><td>0.106667</td><td>13.24701</td><td>0.291286</td><td>0.878652</td></tr><tr><td>50</td><td>50</td><td>1.0</td><td>0.133333</td><td>11.60145</td><td>0.238551</td><td>0.830264</td></tr><tr><td>75</td><td>25</td><td>1.0</td><td>0.166667</td><td>10.22888</td><td>0.186854</td><td>0.770565</td></tr></table>

Tabla 4. Cont.

<table><tr><td>Model</td><td>Solution</td><td>α</td><td>β</td><td>Acc_Clean</td><td>Acc_Protected</td><td>Psnr_Mean</td><td>Ssim_Mean</td><td>Cosine_Flat_Mean</td></tr><tr><td rowspan="10">Swin-b</td><td rowspan="5">noise</td><td>0</td><td>100</td><td>1.0</td><td>0.22</td><td>14.69491</td><td>0.31905</td><td>0.892694</td></tr><tr><td>10</td><td>90</td><td>1.0</td><td>0.166667</td><td>12.81076</td><td>0.298929</td><td>0.856513</td></tr><tr><td>25</td><td>75</td><td>1.0</td><td>0.133333</td><td>10.95512</td><td>0.262094</td><td>0.799997</td></tr><tr><td>50</td><td>50</td><td>1.0</td><td>0.18</td><td>8.781365</td><td>0.190377</td><td>0.698234</td></tr><tr><td>75</td><td>25</td><td>1.0</td><td>0.493333</td><td>7.265331</td><td>0.106383</td><td>0.594237</td></tr><tr><td rowspan="5">replace</td><td>0</td><td>100</td><td>1.0</td><td>0.22</td><td>14.83456</td><td>0.322423</td><td>0.902163</td></tr><tr><td>10</td><td>90</td><td>1.0</td><td>0.146667</td><td>14.20433</td><td>0.316984</td><td>0.896518</td></tr><tr><td>25</td><td>75</td><td>1.0</td><td>0.206667</td><td>13.24701</td><td>0.291286</td><td>0.878652</td></tr><tr><td>50</td><td>50</td><td>1.0</td><td>0.253333</td><td>11.60145</td><td>0.238551</td><td>0.830264</td></tr><tr><td>75</td><td>25</td><td>1.0</td><td>0.52</td><td>10.22888</td><td>0.186854</td><td>0.770565</td></tr><tr><td rowspan="10">ViT-B/16</td><td rowspan="5">noise</td><td>0</td><td>100</td><td>1.0</td><td>0.08</td><td>14.69491</td><td>0.31905</td><td>0.892694</td></tr><tr><td>10</td><td>90</td><td>1.0</td><td>0.106667</td><td>12.81076</td><td>0.298929</td><td>0.856513</td></tr><tr><td>25</td><td>75</td><td>1.0</td><td>0.1</td><td>10.95512</td><td>0.262094</td><td>0.799997</td></tr><tr><td>50</td><td>50</td><td>1.0</td><td>0.153333</td><td>8.781365</td><td>0.190377</td><td>0.698234</td></tr><tr><td>75</td><td>25</td><td>1.0</td><td>0.233333</td><td>7.265331</td><td>0.106383</td><td>0.594237</td></tr><tr><td rowspan="5">replace</td><td>0</td><td>100</td><td>1.0</td><td>0.113333</td><td>14.83456</td><td>0.322423</td><td>0.902163</td></tr><tr><td>10</td><td>90</td><td>1.0</td><td>0.093333</td><td>14.20433</td><td>0.316984</td><td>0.896518</td></tr><tr><td>25</td><td>75</td><td>1.0</td><td>0.12</td><td>13.24701</td><td>0.291286</td><td>0.878652</td></tr><tr><td>50</td><td>50</td><td>1.0</td><td>0.153333</td><td>11.60145</td><td>0.238551</td><td>0.830264</td></tr><tr><td>75</td><td>25</td><td>1.0</td><td>0.16</td><td>10.22888</td><td>0.186854</td><td>0.770565</td></tr><tr><td rowspan="10">resnet50</td><td rowspan="5">noise</td><td>0</td><td>100</td><td>0.86</td><td>0.08</td><td>14.69491</td><td>0.31905</td><td>0.892694</td></tr><tr><td>10</td><td>90</td><td>0.86</td><td>0.106667</td><td>12.81076</td><td>0.298929</td><td>0.856513</td></tr><tr><td>25</td><td>75</td><td>0.86</td><td>0.1</td><td>10.95512</td><td>0.262094</td><td>0.799997</td></tr><tr><td>50</td><td>50</td><td>0.86</td><td>0.1</td><td>8.781365</td><td>0.190377</td><td>0.698234</td></tr><tr><td>75</td><td>25</td><td>0.86</td><td>0.1</td><td>7.265331</td><td>0.106383</td><td>0.594237</td></tr><tr><td rowspan="5">replace</td><td>0</td><td>100</td><td>0.86</td><td>0.073333</td><td>14.83456</td><td>0.322423</td><td>0.902163</td></tr><tr><td>10</td><td>90</td><td>0.86</td><td>0.153333</td><td>14.20433</td><td>0.316984</td><td>0.896518</td></tr><tr><td>25</td><td>75</td><td>0.86</td><td>0.186667</td><td>13.24701</td><td>0.291286</td><td>0.878652</td></tr><tr><td>50</td><td>50</td><td>0.86</td><td>0.126667</td><td>11.60145</td><td>0.238551</td><td>0.830264</td></tr><tr><td>75</td><td>25</td><td>0.86</td><td>0.1</td><td>10.22888</td><td>0.186854</td><td>0.770565</td></tr><tr><td rowspan="10">EfficientNet</td><td rowspan="5">noise</td><td>0</td><td>100</td><td>0.993333</td><td>0.093333</td><td>14.69491</td><td>0.31905</td><td>0.892694</td></tr><tr><td>10</td><td>90</td><td>0.993333</td><td>0.113333</td><td>12.81076</td><td>0.298929</td><td>0.856513</td></tr><tr><td>25</td><td>75</td><td>0.993333</td><td>0.12</td><td>10.95512</td><td>0.262094</td><td>0.799997</td></tr><tr><td>50</td><td>50</td><td>0.993333</td><td>0.16</td><td>8.781365</td><td>0.190377</td><td>0.698234</td></tr><tr><td>75</td><td>25</td><td>0.993333</td><td>0.186667</td><td>7.265331</td><td>0.106383</td><td>0.594237</td></tr><tr><td rowspan="5">replace</td><td>0</td><td>100</td><td>0.993333</td><td>0.093333</td><td>14.83456</td><td>0.322423</td><td>0.902163</td></tr><tr><td>10</td><td>90</td><td>0.993333</td><td>0.146667</td><td>14.20433</td><td>0.316984</td><td>0.896518</td></tr><tr><td>25</td><td>75</td><td>0.993333</td><td>0.126667</td><td>13.24701</td><td>0.291286</td><td>0.878652</td></tr><tr><td>50</td><td>50</td><td>0.993333</td><td>0.16</td><td>11.60145</td><td>0.238551</td><td>0.830264</td></tr><tr><td>75</td><td>25</td><td>0.993333</td><td>0.14</td><td>10.22888</td><td>0.186854</td><td>0.770565</td></tr></table>

Tabla 4. Cont.

<table><tr><td>Model</td><td>Solution</td><td>α</td><td>β</td><td>Acc_Clean</td><td>Acc_Protected</td><td>Psnr_Mean</td><td>Ssim_Mean</td><td>Cosine_Flat_Mean</td></tr><tr><td rowspan="10">MobileNet</td><td rowspan="5">noise</td><td>0</td><td>100</td><td>1.0</td><td>0.12</td><td>14.69491</td><td>0.31905</td><td>0.892694</td></tr><tr><td>10</td><td>90</td><td>1.0</td><td>0.126667</td><td>12.81076</td><td>0.298929</td><td>0.856513</td></tr><tr><td>25</td><td>75</td><td>1.0</td><td>0.1</td><td>10.95512</td><td>0.262094</td><td>0.799997</td></tr><tr><td>50</td><td>50</td><td>1.0</td><td>0.1</td><td>8.781365</td><td>0.190377</td><td>0.698234</td></tr><tr><td>75</td><td>25</td><td>1.0</td><td>0.1</td><td>7.265331</td><td>0.106383</td><td>0.594237</td></tr><tr><td rowspan="5">replace</td><td>0</td><td>100</td><td>1.0</td><td>0.12</td><td>14.83456</td><td>0.322423</td><td>0.902163</td></tr><tr><td>10</td><td>90</td><td>1.0</td><td>0.153333</td><td>14.20433</td><td>0.316984</td><td>0.896518</td></tr><tr><td>25</td><td>75</td><td>1.0</td><td>0.166667</td><td>13.24701</td><td>0.291286</td><td>0.878652</td></tr><tr><td>50</td><td>50</td><td>1.0</td><td>0.126667</td><td>11.60145</td><td>0.238551</td><td>0.830264</td></tr><tr><td>75</td><td>25</td><td>1.0</td><td>0.12</td><td>10.22888</td><td>0.186854</td><td>0.770565</td></tr></table>

## 5.7. Evaluación de imágenes protegidas (Evaluation of Protected Images)

La Tabla 2 resume los resultados experimentales cuantitativos obtenidos al aplicar tres variantes de protección PIS (ruido, reemplazo y textura) a los conjuntos de datos ImageNet-100, CIFAR-10 y CIFAR-100 (subconjunto). Las métricas reportadas incluyen el accuracy de clasificación, la confianza de predicción (prediction confidence), la entropía y las medidas de similitud de imagen (SSIM y PSNR), lo que permite un análisis conjunto de la perturbación del aprendizaje y la preservación perceptual. Para ImageNet-100, todas las variantes de protección reducen el accuracy de clasificación a aproximadamente un 9.1%, lo que indica una degradación sustancial del rendimiento de aprendizaje y generalización. En comparación con la protección basada en ruido, las variantes de reemplazo y textura exhiben una menor confianza media y una mayor entropía, lo que sugiere una mayor incertidumbre predictiva. A pesar de esta perturbación, los valores de SSIM y PSNR se mantienen relativamente estables, confirmando que la estructura global y la calidad perceptual de las imágenes se conservan en gran medida tras la protección.

En el caso de CIFAR-10, el accuracy de clasificación colapsa a cero para todas las variantes de protección. Este comportamiento no indica una predicción aleatoria, sino que refleja una incapacidad del modelo para aprender características discriminativas a partir de las imágenes protegidas. Esta interpretación se ve respaldada adicionalmente por los valores de entropía consistentemente altos observados en todas las variantes. Las métricas SSIM y PSNR no se reportan para CIFAR-10. Esto se debe a que, en los experimentos con CIFAR-10, las imágenes protegidas se generaron y consumieron sobre la marcha (on-the-fly) dentro del pipeline de entrenamiento, sin almacenarse como muestras de imágenes emparejadas alineadas con sus contrapartes limpias a resoluciones idénticas. En consecuencia, no fue posible establecer la correspondencia píxel a píxel requerida para el cómputo de SSIM y PSNR.

Para los experimentos con CIFAR-100 (subconjunto), que representan un régimen de datos extremadamente reducido, todas las variantes de protección conducen nuevamente a un colapso total del accuracy. En comparación con ImageNet-100, la confianza promedio se reduce significativamente, mientras que la entropía permanece en un nivel moderado, lo que indica una fuerte perturbación del aprendizaje bajo condiciones de datos limitados. Al mismo tiempo, los valores de SSIM y PSNR permanecen relativamente altos, demostrando que la similitud perceptual se preserva incluso cuando el tamaño del conjunto de datos está severamente restringido. En general, estos resultados demuestran cuantitativamente que PIS induce de manera consistente el colapso del aprendizaje en conjuntos de datos de variadas resoluciones y escalas, manteniendo la integridad perceptual desde la perspectiva humana.

## 6. Discusión

Una observación importante a partir de los resultados experimentales es que el accuracy de clasificación en imágenes protegidas (acc\_protected) disminuye sustancialmente incluso cuando la cosine similarity entre las representaciones de características de las imágenes limpias y protegidas se mantiene relativamente alta. Este resultado sugiere que el mecanismo de protección propuesto no induce distorsiones arbitrarias o globales en el espacio de características. En cambio, interfiere selectivamente con componentes de características localizadas que son cruciales para la decisión de clasificación final [21,24].

En general, los clasificadores de imágenes basados en deep learning dependen de un subconjunto limitado de características altamente discriminativas incrustadas en espacios de representación de alta dimensión. Mediante el uso de Layer-Wise Relevance Propagation (LRP), la solución propuesta Predictable-Image Solution (PIS) identifica explícitamente estas características críticas y aplica modificaciones dirigidas a las correspondientes regiones de la imagen. Como resultado, la orientación global de los vectores de características se conserva en gran medida, lo que conduce a una alta cosine similarity, mientras que la información discriminativa requerida para cruzar las fronteras de decisión se perturba eficazmente. Esto explica por qué las métricas basadas únicamente en la similitud son insuficientes para capturar la degradación en el rendimiento de clasificación.

En comparación con los enfoques de protección convencionales basados en la inyección global de ruido o en la distorsión uniforme, PIS logra un trade-off favorable entre la fidelidad perceptual y la perturbación del aprendizaje. Las imágenes protegidas permanecen visualmente similares a las originales desde una perspectiva humana, mientras que el rendimiento de inferencia de los modelos pre-trained se degrada significativamente. Esta característica subraya la idoneidad de PIS como un mecanismo práctico de copyright protection en escenarios del mundo real, donde preservar la calidad visual suele ser tan importante como prevenir la explotación no autorizada del modelo.

## 7. Limitaciones del estudio

A pesar de la efectividad general del enfoque propuesto, deben reconocerse varias limitaciones. En primer lugar, bajo ciertas configuraciones de protección, se observó un incremento inesperado en el accuracy de clasificación promedio en imágenes protegidas. Esto indica que una mayor protección no conduce necesariamente a una degradación monótona del rendimiento de clasificación. Por el contrario, la interacción entre el reemplazo estructurado de características y la arquitectura del modelo puede exhibir un comportamiento no lineal. En particular, cuando se modifican regiones locales altamente discriminativas, algunos modelos pueden compensar apoyándose en pistas estadísticas o globales alternativas.

Este fenómeno es especialmente relevante para las arquitecturas basadas en Transformer, donde los mecanismos de global self-attention permiten una agregación holística de la información. Bajo configuraciones específicas, la redistribución de la atención desencadenada por el proceso de protección puede mejorar involuntariamente la separabilidad entre clases en lugar de suprimirla. Estos resultados sugieren que la adaptación a nivel de representación puede compensar parcialmente la perturbación localizada, enfatizando la necesidad de estrategias de protección conscientes de la arquitectura (architecture-aware).

En segundo lugar, este estudio empleó proporciones de protección fijas, y se excluyeron las configuraciones que implicaban el reemplazo total de las regiones con relevancia positiva. Una proporción de protección del 100% resultaría en la pérdida completa de la semántica visual debido a la sustitución total de la imagen, lo que la hace impracticable para aplicaciones en el mundo real. Sin embargo, el uso de proporciones fijas podría no explorar plenamente el trade-off óptimo entre la fuerza de protección y la preservación semántica. En consecuencia, algunas configuraciones podrían no explotar al máximo el potencial de protección, mientras que otras podrían permitir una recuperación parcial del rendimiento de clasificación. En general, estos hallazgos indican que una copyright protection eficaz en sistemas de deep learning requiere una consideración más profunda de la dinámica de las representaciones y de los comportamientos de adaptación dependientes del modelo.

## 8. Conclusiones

En este estudio, propusimos la Predictable-Image Solution (PIS), un marco de copyright protection que modifica selectivamente regiones visualmente críticas identificadas a través de Layer-Wise Relevance Propagation (LRP). A diferencia de los métodos de protección convencionales basados en la inyección indiscriminada de ruido o en la distorsión global, PIS introduce transformaciones estructuradas y conscientes de la decisión que apuntan directamente a las características discriminativas aprendidas durante el entrenamiento del modelo.

Los resultados experimentales demuestran que el enfoque propuesto degrada significativamente el accuracy de clasificación de los modelos pre-trained mientras mantiene una cosine similarity relativamente alta entre las representaciones de características de las imágenes originales y protegidas. Esto indica que PIS perturba los procesos de decisión del modelo sin depender de una degradación perceptual excesiva.

Asimismo, la discrepancia observada entre la similitud en el espacio de características y el rendimiento de clasificación resalta la importancia de dirigirse a regiones relevantes para la decisión en lugar de limitarse a alterar la apariencia visual. Los comportamientos dependientes de la arquitectura identificados en ciertos modelos sugieren además que la efectividad de la protección está estrechamente ligada a los mecanismos de representación subyacentes. En general, PIS proporciona una estrategia práctica y model-agnostic para proteger imágenes sujetas a derechos de autor en pipelines modernos de machine learning y establece las bases para una protección de datos impulsada por la explicabilidad.

## 9. Trabajos futuros

Las investigaciones futuras se concentrarán en el avance de estrategias de protección adaptativas que ajusten dinámicamente la fuerza de protección según la arquitectura del modelo y las características del conjunto de datos. El análisis de los patrones de redistribución de relevancia puede permitir el diseño de mecanismos de enmascaramiento architecture-aware que logren una degradación más consistente a través de diversas familias de modelos.

Otra dirección importante implica el desarrollo de técnicas de optimización automatizada de parámetros que equilibren la efectividad de la protección con las restricciones de calidad perceptual. La integración de métricas de similitud perceptual en el marco de optimización podría permitir la determinación dinámica de la intensidad de reemplazo mientras se preserva la coherencia semántica.

Además, el marco propuesto se extenderá más allá de la clasificación de imágenes hacia paradigmas de aprendizaje más amplios, incluyendo modelos generativos, self-supervised learning y sistemas multimodales. Investigar el comportamiento de la protección guiada por relevancia dentro de modelos de IA generativa a gran escala será crucial para evaluar su aplicabilidad práctica en escenarios del mundo real.

Este estudio evalúa principalmente la efectividad del marco de protección PIS en modelos de clasificación de imágenes. Sin embargo, las cuestiones de derechos de autor también son cada vez más relevantes para los modelos generativos tales como los modelos de difusión y las GAN, que se entrenan sobre conjuntos de datos de imágenes a gran escala. En tales escenarios, perturbar selectivamente las regiones visuales discriminativas puede limitar la capacidad de los modelos generativos para reproducir directamente características visuales específicas de las imágenes protegidas. Investigar la aplicabilidad y efectividad de PIS en el entrenamiento de modelos generativos sigue siendo, por tanto, una dirección importante para trabajos futuros.

**Contribuciones de los autores:** Conceptualización, Y.P.; metodología, S.K.; software, S.K.; validación, S.K. y Y.P.; análisis formal, S.K.; investigación, Y.P.; curación de datos, Y.P.; redacción—preparación del borrador original, S.K.; redacción—revisión y edición, Y.P. e I.J.; visualización, S.K.; supervisión, I.J.; administración del proyecto, I.J. Todos los autores han leído y acordado la versión publicada del manuscrito.

**Financiamiento:** Esta investigación no recibió financiamiento externo.

**Declaración de la Junta de Revisión Institucional:** No aplicable.

**Declaración de consentimiento informado:** No aplicable.

**Declaración de disponibilidad de datos:** Los conjuntos de datos utilizados en este estudio están disponibles públicamente. ImageNet-100 se construyó a partir del conjunto de datos ImageNet, y CIFAR-10 y CIFAR-100 están disponibles públicamente en sus repositorios oficiales.

**Conflictos de interés:** Los autores declaran no tener conflictos de interés.

## Referencias

1. Tramèr, F.; Zhang, F.; Juels, A.; Reiter, M.K.; Ristenpart, T. Stealing machine learning models via prediction {APIs}. In 25th USENIX Security Symposium (USENIX Security 16); USENIX Association: Berkeley, CA, USA, 2016; pp. 601–618.

2. Bach, S.; Binder, A.; Montavon, G.; Klauschen, F.; Müller, K.R.; Samek, W. On pixel-wise explanations for non-linear classifier decisions by layer-wise relevance propagation. PLoS ONE 2015, 10, e0130140. [[CrossRef](http://dx.doi.org/10.1371/journal.pone.0130140)] [[PubMed](http://www.ncbi.nlm.nih.gov/pubmed/26161953)]
3. Montavon, G.; Samek, W.; Müller, K.R. Methods for interpreting and understanding deep neural networks. Digit. Signal Process. 2018, 73, 1–15. [[CrossRef](http://dx.doi.org/10.1016/j.dsp.2017.10.011)]
4. Zhang, J.; Gu, Z.; Jang, J.; Wu, H.; Stoecklin, M.P.; Huang, H.; Molloy, I. Protecting intellectual property of deep neural networks with watermarking. In Proceedings of the 2018 on Asia Conference on Computer and Communications Security; Association for Computing Machinery: New York, NY, USA, 2018; pp. 159–172.
5. Editing, B.S. Photoshop Tutorial-How to Remove Watermark. 2024. Available online: [https://youtu.be/cJIQyzwOZeY?si=0GATDqwjHXAR2cfM](https://youtu.be/cJIQyzwOZeY?si=0GATDqwjHXAR2cfM) (accessed on 7 October 2025 ).
6. Criminisi, A.; Pérez, P.; Toyama, K. Region filling and object removal by exemplar-based image inpainting. IEEE Trans. Image Process. 2004, 13, 1200–1212. [[CrossRef](http://dx.doi.org/10.1109/TIP.2004.833105)] [[PubMed](http://www.ncbi.nlm.nih.gov/pubmed/15449582)]
7. Shan, S.; Wenger, E.; Zhang, J.; Li, H.; Zheng, H.; Zhao, B.Y. Fawkes: Protecting privacy against unauthorized deep learning models. In 29th USENIX Security Symposium (USENIX Security 20); USENIX Association: Berkeley, CA, USA, 2020; pp. 1589–1604.
8. Shan, S.; Cryan, J.; Wenger, E.; Zheng, H.; Hanocka, R.; Zhao, B.Y. Glaze: Protecting artists from style mimicry by {Text-to-Image} models. In 32nd USENIX Security Symposium (USENIX Security 23); USENIX Association: Berkeley, CA, USA, 2023; pp. 2187–2204.
9. Liu, Y.; Fan, C.; Dai, Y.; Chen, X.; Zhou, P.; Sun, L. Metacloak: Preventing unauthorized subject-driven text-to-image diffusionbased synthesis via meta-learning. In 2024 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR); IEEE: Piscataway, NJ, USA, 2024; pp. 24219–24228.
10. Selvaraju, R.R.; Cogswell, M.; Das, A.; Vedantam, R.; Parikh, D.; Batra, D. Grad-cam: Visual explanations from deep networks via gradient-based localization. In 2017 IEEE International Conference on Computer Vision (ICCV); IEEE: Piscataway, NJ, USA, 2017; pp. 618–626.
11. Ribeiro, M.T.; Singh, S.; Guestrin, C. “Why should i trust you?” Explaining the predictions of any classifier. In Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining; Association for Computing Machinery: New York, NY, USA, 2016; pp. 1135–1144.
12. Lundberg, S.M.; Lee, S.I. A unified approach to interpreting model predictions. In Advances in Neural Information Processing Systems 30; NeurIPS: San Diego, CA, USA, 2017; Volume 30.
13. Sundararajan, M.; Taly, A.; Yan, Q. Axiomatic attribution for deep networks. In Proceedings of the 34th International Conference on Machine Learning; PMLR: Cambridge, MA, USA, 2017; pp. 3319–3328.
14. Montavon, G.; Binder, A.; Lapuschkin, S.; Samek, W.; Müller, K.R. Layer-wise relevance propagation: An overview. In Explainable AI: Interpreting, Explaining and Visualizing Deep Learning; Springer: Cham, Switzerland, 2019; pp. 193–209.
15. Montavon, G.; Lapuschkin, S.; Binder, A.; Samek, W.; Müller, K.R. Explaining nonlinear classification decisions with deep taylor decomposition. Pattern Recognit. 2017, 65, 211–222. [[CrossRef](http://dx.doi.org/10.1016/j.patcog.2016.11.008)]
16. Greenshed. Ceiling Boss over the Centre of the Nave in Carlisle Cathedral. 2009. Available online: [https://commons.wikimedia.org/wiki/File:Ceiling\_boss\_over\_the\_centre\_of\_the\_nave\_in\_Carlisle\_Cathedral.jpg](https://commons.wikimedia.org/wiki/File:Ceiling_boss_over_the_centre_of_the_nave_in_Carlisle_Cathedral.jpg) (accessed on 7 October 2025).
17. Dosovitskiy, A. An image is worth 16x16 words: Transformers for image recognition at scale. arXiv 2020, arXiv:2010.11929.
18. Russakovsky, O.; Deng, J.; Su, H.; Krause, J.; Satheesh, S.; Ma, S.; Huang, Z.; Karpathy, A.; Khosla, A.; Bernstein, M.; et al. ImageNet Large Scale Visual Recognition Challenge. Int. J. Comput. Vis. (IJCV) 2015, 115, 211–252. [[CrossRef](http://dx.doi.org/10.1007/s11263-015-0816-y)]
19. Tian, Y.; Krishnan, D.; Isola, P. Contrastive multiview coding. In Proceedings of the Computer Vision–ECCV 2020: 16th European Conference, Glasgow, UK, August 23–28. 2020, Proceedings, Part XI 16; Springer: Cham, Switzerland, 2020; pp. 776–794.
20. Krizhevsky, A.; Hinton, G. Learning Multiple Layers of Features from Tiny Images. 2009. Available online: [http://www.cs.utoronto.ca/\~kriz/learning-features-2009-TR.pdf](http://www.cs.utoronto.ca/~kriz/learning-features-2009-TR.pdf) (accessed on 7 October 2025).
21. Liu, Z.; Lin, Y.; Cao, Y.; Hu, H.; Wei, Y.; Zhang, Z.; Lin, S.; Guo, B. Swin transformer: Hierarchical vision transformer using shifted windows. In 2021 IEEE/CVF International Conference on Computer Vision (ICCV); IEEE: Piscataway, NJ, USA, 2021; pp. 10012–10022.
22. Fawzi, A.; Fawzi, O.; Frossard, P. Analysis of classifiers’ robustness to adversarial perturbations. Mach. Learn. 2018, 107, 481–508. [[CrossRef](http://dx.doi.org/10.1007/s10994-017-5663-3)]
23. Ilyas, A.; Santurkar, S.; Tsipras, D.; Engstrom, L.; Tran, B.; Madry, A. Adversarial examples are not bugs, they are features. In Advances in Neural Information Processing Systems 32; NeurIPS: San Diego, CA, USA, 2019; Volume 32.
24. Touvron, H.; Cord, M.; Douze, M.; Massa, F.; Sablayrolles, A.; Jégou, H. Training data-efficient image transformers & distillation through attention. In Proceedings of the 38th International Conference on Machine Learning; PMLR: Cambridge, MA, USA, 2021; pp. 10347–10357.
25. Cordonnier, J.B.; Loukas, A.; Jaggi, M. On the relationship between self-attention and convolutional layers. arXiv 2019, arXiv:1911.03584.

**Descargo de responsabilidad/Nota del editor:** Las declaraciones, opiniones y datos contenidos en todas las publicaciones pertenecen exclusivamente al autor(es) y contribuyente(s) individual(es) y no a MDPI y/o al editor(es). MDPI y/o el editor(es) declinan toda responsabilidad por cualquier daño a personas o propiedades que resulte de cualesquiera ideas, métodos, instrucciones o productos a los que se haga referencia en el contenido.


<details>
<summary>Páginas originales para cotejo</summary>

### Página 1

![Página 1](original-pages/page-01.png)

### Página 2

![Página 2](original-pages/page-02.png)

### Página 3

![Página 3](original-pages/page-03.png)

### Página 4

![Página 4](original-pages/page-04.png)

### Página 5

![Página 5](original-pages/page-05.png)

### Página 6

![Página 6](original-pages/page-06.png)

### Página 7

![Página 7](original-pages/page-07.png)

### Página 8

![Página 8](original-pages/page-08.png)

### Página 9

![Página 9](original-pages/page-09.png)

### Página 10

![Página 10](original-pages/page-10.png)

### Página 11

![Página 11](original-pages/page-11.png)

### Página 12

![Página 12](original-pages/page-12.png)

### Página 13

![Página 13](original-pages/page-13.png)

### Página 14

![Página 14](original-pages/page-14.png)

### Página 15

![Página 15](original-pages/page-15.png)

### Página 16

![Página 16](original-pages/page-16.png)

### Página 17

![Página 17](original-pages/page-17.png)

### Página 18

![Página 18](original-pages/page-18.png)

</details>

