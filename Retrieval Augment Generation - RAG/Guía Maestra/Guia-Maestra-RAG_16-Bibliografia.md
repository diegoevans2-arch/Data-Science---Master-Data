---
title: "Tomo 16 — Bibliografía"
tags: [rag, bibliografia, referencias, transversal, verificacion]
audiencias: [tecnico, puente, ejecutivo]
tomo: 16
version: 1.1
updated: 2026-09-05
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 16 — Bibliografía

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|Tomo 15 · Glosario ejecutivo]]

---

> [!info] 📌 Sobre esta bibliografía
> Todas las obras citadas en los Tomos 01–14, con sus datos de publicación completos. Las citas en el texto usan el formato `(Autor, Año)`.
>
> ✅ **52/52 obras verificadas contra fuente primaria (2026-07-28)** — cada una con evidencia y DOI/URL. La auditoría corrigió **4 errores** y precisó **6 fichas** más; el detalle está en la sección 14, porque los errores encontrados son didácticos en sí mismos.
>
> ➕ **67 fichas incorporadas el 2026-09-05** (§15–§18): las 37 del Tomo 12 y las 12 del Tomo 13 (verificadas al escribirlos, el 2026-07-28/29, pero nunca consolidadas aquí), las 7 del Tomo 14 y las 11 adiciones de la revisión del 2026-08-28 — estas 18 últimas verificadas el 2026-09-05, con **4 errores corregidos en el Tomo 14, una referencia inexistente eliminada del Tomo 04 y un título corregido en el Tomo 06**. **Total: 119 fichas.**
>
> El **§5 de las [[Instrucciones|Instrucciones]]** exige que toda afirmación de fuente externa tenga bibliografía verificable. Este tomo es donde eso se comprueba.

> [!warning] ⚠️ Tres apellidos que se citan mal con frecuencia
> - **Spärck Jones, K.** — apellido **compuesto**. Nunca "Jones, K. S.". El artículo de 1972 se imprimió sin diéresis ("SPARCK JONES"), pero la forma que ella adoptó y que usa la convención académica es *Spärck Jones*.
> - **Büttcher, S.** — con diéresis. ACM DL lo translitera como "Buettcher"; la forma impresa en el paper es *Büttcher*.
> - **Oğuz, B.** — con ğ turca. ACL Anthology lo normaliza a "Oguz".

---

## 1. Fuente primaria

- **DeepLearning.AI.** *Retrieval Augmented Generation (RAG)* (Coursera / Universidad San Sebastián). Módulos 1–5: transcripciones de las lecciones, Ungraded Labs, assignments graded C1M1–C1M5 y quizzes. — **Fuente primaria de los Tomos 01–11.**

> [!note] Por qué esta entrada no lleva ✅
> Es material de curso tras autenticación, no una obra publicada y verificable de forma independiente. Su verificación es de otro tipo: **contrastar contra el material entregado y ejecutar el código**, que es lo que hacen los tomos (ver los hallazgos §4.3 del [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10]] y §4.3 del [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11]]).

---

## 2. Papers fundacionales de RAG

- Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., Küttler, H., Lewis, M., Yih, W., Rocktäschel, T., Riedel, S., & Kiela, D. (2020). "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks". *Advances in Neural Information Processing Systems 33 (NeurIPS 2020)*. arXiv:2005.11401. ✅ *Verificada 2026-07-28 (papers.nips.cc; dblp.org). **El paper fundacional.** Son 12 autores; el "et al." de los tomos los ocultaba. Nota: las páginas 9459–9474 que circulan proceden del volumen impreso de Curran y no se pudieron confirmar en fuente primaria — se omiten.* — Tomos [[Guia-Maestra-RAG_01-Introduccion-a-RAG|01]], [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|02]], [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09]]
- Gao, Y., Xiong, Y., Gao, X., Jia, K., Pan, J., Bi, Y., Dai, Y., Sun, J., Wang, M., & Wang, H. (2023). *Retrieval-Augmented Generation for Large Language Models: A Survey*. arXiv:2312.10997. ✅ *Verificada 2026-07-28 (arxiv.org). **Sigue siendo preprint**: el campo* journal reference *está vacío y* comments *dice literalmente "Ongoing Work". No corresponde asignarle venue. Cinco versiones (v1 dic-2023 → v5 mar-2024); si se cita contenido de la v5, conviene indicar la versión.* — Tomo [[Guia-Maestra-RAG_01-Introduccion-a-RAG|01]] (taxonomía Naive / Advanced / Modular RAG)

---

## 3. LLMs, transformers y tokenización

- Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, Ł., & Polosukhin, I. (2017). "Attention Is All You Need". *Advances in Neural Information Processing Systems 30 (**NIPS** 2017)*, 5998–6008. arXiv:1706.03762. ✅ *Verificada 2026-07-28 (papers.nips.cc; dblp.org). **Corrección: el venue es NIPS, no NeurIPS** — el rebranding ocurrió en 2018, un año después de este paper.* — Tomos [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|02]], [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|08]]
- Sennrich, R., Haddow, B., & Birch, A. (2016). "Neural Machine Translation of Rare Words with Subword Units". *Proceedings of the 54th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, 1715–1725. DOI 10.18653/v1/P16-1162. ✅ *Verificada 2026-07-28 (ACL Anthology).* — Tomo [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|02]] (Byte-Pair Encoding)
- Liu, N. F., Lin, K., Hewitt, J., Paranjape, A., Bevilacqua, M., Petroni, F., & Liang, P. (**2024**). "Lost in the Middle: How Language Models Use Long Contexts". *Transactions of the Association for Computational Linguistics*, 12, 157–173. DOI 10.1162/tacl_a_00638. ✅ *Verificada 2026-07-28 (ACL Anthology). **Corrección: el año es 2024, no 2023** — 2023 es el preprint de arXiv; la publicación en TACL es de 2024.* — Tomos [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|02]], [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|08]]
- Ji, Z., Lee, N., Frieske, R., Yu, T., Su, D., Xu, Y., Ishii, E., Bang, Y. J., Madotto, A., & Fung, P. (2023). "Survey of Hallucination in Natural Language Generation". *ACM Computing Surveys*, 55(12), artículo 248, 1–38. DOI 10.1145/3571730. ✅ *Verificada 2026-07-28 (dblp.org + DOI; ACM DL bloquea el acceso automatizado). Se añadió el número de artículo, que en ACM CSUR es necesario para localizar la obra.* — Tomos [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|02]], [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09]]

---

## 4. Information retrieval clásico

- Manning, C. D., Raghavan, P., & Schütze, H. (2008). *Introduction to Information Retrieval*. Cambridge University Press. ISBN 0521865719. ✅ *Verificada 2026-07-28 (sitio oficial del libro en Stanford NLP, con texto completo libre). **El texto canónico de IR.*** — Tomos [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|02]], [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|03]], [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09]]
- Spärck Jones, K. (1972). "A Statistical Interpretation of Term Specificity and Its Application in Retrieval". *Journal of Documentation*, 28(1), 11–21. DOI 10.1108/eb026526. ✅ *Verificada 2026-07-28 (Emerald; bibliografía de Manning et al.). El paper que introduce el **IDF**. Existe una reimpresión de 2004 en el mismo journal, 60(5), 493–502, con datos distintos.* — Tomo [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|03]]
- Robertson, S. E., Walker, S., Jones, S., Hancock-Beaulieu, M. M., & Gatford, M. (1995). "Okapi at TREC-3". En D. K. Harman (Ed.), *Overview of the Third Text REtrieval Conference (TREC-3)*, NIST Special Publication **500-225**, 109–126. Gaithersburg, MD: National Institute of Standards and Technology. ✅ *Verificada 2026-07-28 (trec.nist.gov, PDF primario descargado; dblp.org). **El origen de BM25.*** Ver la nota de abajo. — Tomo [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|03]]

> [!note] Dos precisiones sobre la referencia de Okapi
> **① El año 1995 es correcto.** Se sospechó que los proceedings fueran de 1996 y **la sospecha resultó infundada**: la conferencia TREC-3 se celebró en noviembre de **1994** y sus proceedings se publicaron en **1995**. El "1996" corresponde a **TREC-4** (celebrada en 1995, proceedings en octubre de 1996) — es una confusión de un ciclo completo. Coexisten dos convenciones legítimas: 1995 (año de los proceedings, la que usa NIST en su propia bibliografía y la recomendada) y 1994 (año de la conferencia, la que usa DBLP).
>
> **② El sitio de NIST tiene una errata.** La cabecera de `trec.nist.gov/pubs/trec3/t3_proceedings.html` dice "NIST Special Publication **500-226**". Es un typo del sitio web: el índice oficial y la base de publicaciones de NIST dan **500-225**, y la SP 500-226 es en realidad un documento sin relación (*Self Monitoring Accounting Systems*). **Citar 500-225.**

- Robertson, S., & Zaragoza, H. (2009). "The Probabilistic Relevance Framework: BM25 and Beyond". *Foundations and Trends in Information Retrieval*, 3(4), 333–389. DOI 10.1561/1500000019. ✅ *Verificada 2026-07-28 (PDF oficial descargado; la portada editorial imprime literalmente "Vol. 3, No. 4 (2009) 333-389"). El tratamiento formal de BM25 y sus hiperparámetros.* — Tomo [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|03]]
- Gollapudi, S., Karia, N., Sivashankar, V., Krishnaswamy, R., Begwani, N., Raz, S., Lin, Y., Zhang, Y., Mahapatro, N., Srinivasan, P., Singh, A., & Simhadri, H. V. (2023). "Filtered-DiskANN: Graph Algorithms for Approximate Nearest Neighbor Search with Filters". *Proceedings of the ACM Web Conference 2023 (WWW '23)*, 3406–3416. DOI 10.1145/3543507.3583552. ✅ *Verificada 2026-07-28 (PDF del autor senior con el bloque "ACM Reference Format"; dblp.org). Los 12 autores confirmados.* — Tomo [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|03]] (dificultad de combinar filtros con búsqueda vectorial)

---

## 5. Embeddings y semantic search

- Mikolov, T., Chen, K., Corrado, G., & Dean, J. (2013). *Efficient Estimation of Word Representations in Vector Space*. 1st International Conference on Learning Representations (ICLR 2013), **Workshop Track**. arXiv:1301.3781. ✅ *Verificada 2026-07-28 (arxiv.org; dblp.org XML). **word2vec.** Aviso: varias bases secundarias citan "pp. 1-12" — eso es el paginado del PDF de arXiv, no de unas actas. El workshop track de ICLR 2013 **no tuvo actas paginadas ni DOI**; no usar esas páginas.* — Tomo [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|04]]
- Beyer, K. S., Goldstein, J., Ramakrishnan, R., & Shaft, U. (1999). "When Is 'Nearest Neighbor' Meaningful?". *Database Theory — ICDT '99, 7th International Conference*, Lecture Notes in Computer Science 1540, 217–235. Springer. DOI 10.1007/3-540-49257-7_15. ✅ *Verificada 2026-07-28 (dblp.org). Concentración de distancias en alta dimensión — el fundamento de por qué se usa cosine y no euclidiana.* — Tomo [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|04]]
- Reimers, N., & Gurevych, I. (2019). "Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks". *Proceedings of the 2019 Conference on Empirical Methods in Natural Language Processing and the 9th International Joint Conference on Natural Language Processing (**EMNLP-IJCNLP**)*, 3982–3992. DOI 10.18653/v1/D19-1410. ✅ *Verificada 2026-07-28 (ACL Anthology). **Precisión: el venue de 2019 fue conjunto EMNLP-IJCNLP**; citar solo "EMNLP" es impreciso.* — Tomos [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|04]], [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|07]]
- Karpukhin, V., Oğuz, B., Min, S., Lewis, P., Wu, L., Edunov, S., Chen, D., & Yih, W. (2020). "Dense Passage Retrieval for Open-Domain Question Answering". *Proceedings of EMNLP 2020*, 6769–6781. DOI 10.18653/v1/2020.emnlp-main.550. ✅ *Verificada 2026-07-28 (ACL Anthology). Los 8 autores confirmados. Aquí "EMNLP" a secas **sí** es correcto: la edición de 2020 no fue conjunta.* — Tomo [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|04]]
- Schroff, F., Kalenichenko, D., & Philbin, J. (2015). "FaceNet: A Unified Embedding for Face Recognition and Clustering". *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR)*, 815–823. DOI 10.1109/CVPR.2015.7298682. ✅ *Verificada 2026-07-28 (dblp.org; CVF Open Access). El origen del esquema anchor / positive / negative (triplet loss).* — Tomo [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|04]]
- Cormack, G. V., Clarke, C. L. A., & **Büttcher**, S. (2009). "Reciprocal rank fusion outperforms Condorcet and individual rank learning methods". *Proceedings of the 32nd International ACM SIGIR Conference (SIGIR '09)*, 758–759. DOI 10.1145/1571941.1572114. ✅ *Verificada 2026-07-28 (PDF del autor descargado y extraído; dblp.org). **El paper de RRF.** Precisión de apellido: Büttcher, no Buettcher.* Ver la nota crítica de abajo. — Tomo [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|04]]

> [!warning] ⚠️ Sobre el `k = 60` de RRF — una precisión que cambia cómo debe citarse
> El valor `k = 60` **sí aparece en el paper**, pero **no como óptimo demostrado**. El texto dice que *"k = 60 was fixed during a pilot investigation"* y que la elección resultó *"near-optimal, but that the choice was not critical"*. De hecho, en la Tabla 1 del propio paper el MAP máximo cae en **k = 80** (.2147), ligeramente por encima de k = 60 (.2145): la curva es prácticamente plana entre k = 30 y k = 100.
>
> - ✅ Formulación correcta: *"k = 60, el valor fijado por Cormack et al. (2009) en su investigación piloto, donde los autores reportan que la elección no es crítica"*.
> - ❌ Formulación incorrecta: *"k = 60, el valor óptimo determinado por Cormack et al."*
>
> Importa porque el [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]] documenta ese default: conviene que quede claro que es una convención heredada de un piloto, no una constante optimizada.

---

## 6. Vector databases y ANN

- Malkov, Y., Ponomarenko, A., Logvinov, A., & Krylov, V. (2014). "Approximate nearest neighbor algorithm based on navigable small world graphs". *Information Systems*, 45, 61–68. DOI 10.1016/j.is.2013.10.006. ✅ *Verificada 2026-07-28 (Crossref). **NSW**, el antecesor de HNSW. El DOI contiene "2013" (fecha de aceptación); el issue es de septiembre de 2014.* — Tomo [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|05]]
- Malkov, Yu. A., & Yashunin, D. A. (2020). "Efficient and Robust Approximate Nearest Neighbor Search Using Hierarchical Navigable Small World Graphs". *IEEE Transactions on Pattern Analysis and Machine Intelligence*, 42(4), 824–836. DOI 10.1109/TPAMI.2018.2889473. ✅ *Verificada 2026-07-28 (Crossref: volumen, número, páginas y fecha de issue confirmados). **El paper de HNSW.** Se sospechó un error de año y **la cita resultó correcta en todos sus campos**. Las tres fechas que confunden: arXiv:1603.09320 (2016–2018), early access de IEEE (dic-2018, de ahí el "2018" del DOI) e issue formal (abril de 2020 — el año citable).* — Tomo [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|05]]
- Johnson, J., Douze, M., & Jégou, H. (**2021**). "Billion-Scale Similarity Search with GPUs". *IEEE Transactions on Big Data*, 7(3), 535–547. DOI 10.1109/TBDATA.2019.2921572. ✅ *Verificada 2026-07-28 (Crossref; dblp.org). **FAISS. Corrección: el año es 2021, no 2019.*** Ver la nota de abajo. — Tomo [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|05]]

> [!danger] 🚨 El mecanismo del error de año — se repite en toda la bibliografía de IEEE
> El caso de FAISS explica un patrón que aparece varias veces en este tomo. **Tres fechas distintas conviven** y los gestores bibliográficos eligen la equivocada:
>
> | Fecha | Valor | Qué es |
> |---|---|---|
> | arXiv | feb-2017 | Preprint |
> | **Early access** de IEEE | jun-2019 | Publicado en Xplore **antes** de asignarse a un issue. **El DOI queda sellado con ese año** (`TBDATA.2019.…`) |
> | **Issue formal** | jul-**2021** | Vol. 7, núm. 3, pp. 535–547. **Este es el año citable** |
>
> El error nace de copiar el token de año del DOI. Y Semantic Scholar lo agrava: su registro devuelve `year: 2017` (fecha de arXiv) junto con el volumen y las páginas del issue de 2021 — mezclando metadatos de preprint y de publicación en una sola ficha.
>
> **Regla:** cuando una referencia de IEEE lleve un año en el DOI distinto del año citado, hay que ir a Crossref o DBLP y usar la fecha del **issue**.

- Aumüller, M., Bernhardsson, E., & Faithfull, A. (2020). "ANN-Benchmarks: A benchmarking tool for approximate nearest neighbor algorithms". *Information Systems*, 87, artículo 101374. DOI 10.1016/j.is.2019.02.006. ✅ *Verificada 2026-07-28 (Crossref). Se añadió el número de artículo, obligatorio porque este volumen no usa paginación tradicional. El año del issue (enero 2020) es correcto pese al "2019" del DOI.* — Tomo [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|05]]

---

## 7. Chunking

- Anthropic (2024, 19 de septiembre). *Introducing Contextual Retrieval*. [anthropic.com/news/contextual-retrieval](https://www.anthropic.com/news/contextual-retrieval) ✅ *Verificada 2026-07-28 (página oficial leída directamente). Es la técnica que el curso llama* context-aware chunking. *Anuncio corporativo, no publicación académica.* — Tomo [[Guia-Maestra-RAG_06-Chunking|06]]

> [!warning] ⚠️ Cómo citar las cifras de Contextual Retrieval sin distorsionarlas
> Las tres cifras del post se confirmaron literalmente, pero **la métrica hay que nombrarla siempre**: es la **tasa de fallo de recuperación en los top-20 chunks**, sobre un baseline de **5,7 %**.
>
> | Configuración | Reducción del fallo | Absoluto |
> |---|---|---|
> | Contextual Embeddings | −35 % | 5,7 % → 3,7 % |
> | + Contextual BM25 | −49 % | 5,7 % → 2,9 % |
> | + reranking | −67 % | 5,7 % → 1,9 % |
>
> Sin el calificador *"top-20-chunk retrieval failure rate"*, un "67 % de mejora" se malinterpreta como ganancia de accuracy end-to-end, que es un orden de magnitud distinto.

- Chacon, S., & Straub, B. (2014). *Pro Git* (2.ª ed.). Apress. DOI 10.1007/978-1-4842-0076-6. Código fuente AsciiDoc: [github.com/progit/progit2](https://github.com/progit/progit2) ✅ *Verificada 2026-07-28 (repo, git-scm.com y Crossref). **Usado como corpus** del Ungraded Lab 2, no como fuente teórica.* — Tomo [[Guia-Maestra-RAG_06-Chunking|06]]

> [!note] El corpus es un objeto vivo, y eso afecta la reproducibilidad
> El repositorio **no es idéntico al texto impreso de 2014**: git-scm.com indica que la versión del repo *"se ha actualizado con correcciones y adiciones de cientos de colaboradores"*. Para que un lab sea reproducible hay que **citar el commit o la fecha de clonación**, no solo el año de la edición.

---

## 8. Query parsing, re-ranking y late interaction

- Gao, L., Ma, X., Lin, J., & Callan, J. (2023). "Precise Zero-Shot Dense Retrieval without Relevance Labels". *Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, 1762–1777. DOI 10.18653/v1/2023.acl-long.99. arXiv:2212.10496. ✅ *Verificada 2026-07-28 (ACL Anthology, BibTeX oficial). **El paper de HyDE.** Long paper del main track, no findings ni workshop.* — Tomo [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|07]]
- Khattab, O., & Zaharia, M. (2020). "ColBERT: Efficient and Effective Passage Search via Contextualized Late Interaction over BERT". *Proceedings of the 43rd International ACM SIGIR Conference (SIGIR 2020)*, 39–48. DOI 10.1145/3397271.3401075. ✅ *Verificada 2026-07-28 (dblp.org, BibTeX oficial). **ColBERT y el scoring MaxSim.*** — Tomo [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|07]]
- Santhanam, K., Khattab, O., Saad-Falcon, J., Potts, C., & Zaharia, M. (2022). "ColBERTv2: Effective and Efficient Retrieval via Lightweight Late Interaction". *Proceedings of NAACL-HLT 2022*, 3715–3734. DOI 10.18653/v1/2022.naacl-main.272. ✅ *Verificada 2026-07-28 (ACL Anthology).* — Tomo [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|07]]
- Nogueira, R., & Cho, K. (2019). *Passage Re-ranking with BERT*. arXiv:1901.04085. ✅ *Verificada 2026-07-28 (arxiv.org; dblp.org). **Sigue siendo solo preprint** — DBLP lo clasifica bajo "Informal and Other Publications". No corresponde asignarle venue. Estableció el cross-encoder como re-ranker.* — Tomo [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|07]]
- Zaratiana, U., Tomeh, N., Holat, P., & Charnois, T. (2024). "GLiNER: Generalist Model for Named Entity Recognition using Bidirectional Transformer". *Proceedings of NAACL-HLT 2024 (Volume 1: Long Papers)*, 5364–5376. DOI 10.18653/v1/2024.naacl-long.300. ✅ *Verificada 2026-07-28 (ACL Anthology). El modelo de NER que usa el curso.* — Tomo [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|07]]

---

## 9. Generación, sampling y prompting

- Fan, A., Lewis, M., & Dauphin, Y. (2018). "Hierarchical Neural Story Generation". *Proceedings of the 56th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, 889–898. DOI 10.18653/v1/P18-1082. ✅ *Verificada 2026-07-28 (ACL Anthology + PDF primario). **El origen del top-k sampling**, confirmado con evidencia textual del §5.4: "We randomly sample from the k = 10 most likely candidates from this distribution".* — Tomo [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|08]]

> [!note] Un matiz de atribución
> El paper acuña el término *top-k random sampling* y no cita ninguna obra previa para la técnica, así que la atribución **se sostiene**. Pero los autores lo presentan como **decisión metodológica**, no lo reclaman como contribución en el abstract. Formulación segura: *"introducido como 'top-k random sampling' por Fan et al. (2018)"* — que es la atribución canónica de la literatura posterior.

- Holtzman, A., Buys, J., Du, L., Forbes, M., & Choi, Y. (2020). "The Curious Case of Neural Text Degeneration". *International Conference on Learning Representations (ICLR 2020)*. arXiv:1904.09751. ✅ *Verificada 2026-07-28 (dblp.org; OpenReview `rygGQyrFvH`). **Nucleus sampling (`top_p`).** Los 5 autores confirmados; el año del venue es 2020 (el preprint es de 2019).* — Tomos [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|02]], [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|08]]
- Brown, T. B., Mann, B., Ryder, N., Subbiah, M., Kaplan, J. D., Dhariwal, P., et al. (2020). "Language Models are Few-Shot Learners". *Advances in Neural Information Processing Systems 33 (NeurIPS 2020)*, 1877–1901. arXiv:2005.14165. ✅ *Verificada 2026-07-28 (proceedings.neurips.cc; índice del volumen impreso de Curran, que confirma el rango de páginas). 31 autores.* — Tomo [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|08]]
- Wei, J., Wang, X., Schuurmans, D., Bosma, M., Ichter, B., Xia, F., Chi, E. H., Le, Q. V., & Zhou, D. (2022). "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models". *Advances in Neural Information Processing Systems 35 (NeurIPS 2022)*, 24824–24837. arXiv:2201.11903. ✅ *Verificada 2026-07-28 (PDF oficial de NeurIPS + índice de Curran). Los 9 autores confirmados desde el byline del PDF.* — Tomo [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|08]]
- Kojima, T., Gu, S. S., Reid, M., Matsuo, Y., & Iwasawa, Y. (2022). "Large Language Models are Zero-Shot Reasoners". *Advances in Neural Information Processing Systems 35 (NeurIPS 2022)*, 22199–22213. arXiv:2205.11916. ✅ *Verificada 2026-07-28 (proceedings.neurips.cc; dblp.org). **El paper del "Let's think step by step"**, confirmado en el abstract oficial.* — Tomo [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|08]]
- Hendrycks, D., Burns, C., Basart, S., Zou, A., Mazeika, M., Song, D., & Steinhardt, J. (2021). "Measuring Massive Multitask Language Understanding". *International Conference on Learning Representations (ICLR 2021)*. arXiv:2009.03300. ✅ *Verificada 2026-07-28 (arxiv.org; el campo* comments *declara "ICLR 2021"). **MMLU.*** — Tomo [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|08]]
- Chiang, W.-L., Zheng, L., Sheng, Y., Angelopoulos, A. N., Li, T., Li, D., Zhu, B., Zhang, H., Jordan, M., Gonzalez, J. E., & Stoica, I. (2024). "Chatbot Arena: An Open Platform for Evaluating LLMs by Human Preference". *Proceedings of the 41st International Conference on Machine Learning (ICML 2024)*, PMLR 235, 8359–8388. arXiv:2403.04132. ✅ *Verificada 2026-07-28 (proceedings.mlr.press). Los 11 autores confirmados; se añadieron volumen PMLR y páginas.* — Tomo [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|08]]

---

## 10. Evaluación, hallucinations y agentic RAG

- Es, S., James, J., Espinosa Anke, L., & Schockaert, S. (**2024**). "RAGAs: Automated Evaluation of Retrieval Augmented Generation". *Proceedings of the 18th Conference of the European Chapter of the Association for Computational Linguistics: **System Demonstrations***, 150–158. DOI 10.18653/v1/2024.eacl-demo.16. arXiv:2309.15217. ✅ *Verificada 2026-07-28 (ACL Anthology). **Tres correcciones:** el año es 2024 (2023 es el preprint); le faltaba el venue por completo; y la Anthology registra el título como "RAGAs", no "RAGAS". Nota de apellido: la Anthology usa "Espinosa Anke" sin guion.* — Tomos [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09]], [[Guia-Maestra-RAG_10-RAG-en-Produccion|10]]
- Cohen-Wang, B., Shah, H., Georgiev, K., & Mądry, A. (2024). "ContextCite: Attributing Model Generation to Context". *Advances in Neural Information Processing Systems 37 (NeurIPS 2024)*. arXiv:2409.00729. ✅ *Verificada 2026-07-28 (proceedings.neurips.cc). NeurIPS no publica paginación para este volumen, así que se omite. Grafía canónica: Mądry, con ogonek.* — Tomo [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09]]
- Gao, T., Yen, H., Yu, J., & Chen, D. (2023). "Enabling Large Language Models to Generate Text with Citations". *Proceedings of EMNLP 2023*, 6465–6488. DOI 10.18653/v1/2023.emnlp-main.398. ✅ *Verificada 2026-07-28 (ACL Anthology). **El benchmark ALCE**, confirmado en el abstract: "ALCE, the first benchmark for Automatic LLMs' Citation Evaluation".* — Tomo [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09]]
- Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C. L., Mishkin, P., et al. (2022). "Training language models to follow instructions with human feedback". *Advances in Neural Information Processing Systems 35 (NeurIPS 2022)*, 27730–27744. ✅ *Verificada 2026-07-28 (proceedings.neurips.cc; dblp.org). **InstructGPT.** 20 autores.* — Tomo [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09]]
- Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., Wang, L., & Chen, W. (**2022**). "LoRA: Low-Rank Adaptation of Large Language Models". *International Conference on Learning Representations (ICLR 2022)*. arXiv:2106.09685. ✅ *Verificada 2026-07-28 (arxiv.org; dblp.org). **Corrección de coherencia:** la entrada original mezclaba "2021" con la nota "(ICLR 2022)". Si el venue es ICLR, el año es 2022; si se cita el preprint, es 2021 sin venue. Se adopta la forma canónica ICLR 2022.* — Tomo [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09]]
- Schluntz, E., & Zhang, B. (2024, 19 de diciembre). *Building effective agents*. Anthropic. [anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents) ✅ *Verificada 2026-07-28 (página oficial). Los cinco patrones que documenta el Tomo 09 se confirmaron: prompt chaining, routing, parallelization, orchestrator-workers y evaluator-optimizer. Anuncio corporativo, no publicación académica. Nota: la página firma literalmente "Erik S. and Barry Zhang" — el apellido completo del primer autor no consta en la fuente primaria.* — Tomo [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09]]

---

## 11. Producción: observability y security

*Estas seis se verificaron al escribir los Tomos 10 y 11; se reproducen aquí para que la bibliografía sea completa.*

- Reid, E. (2024, 30 de mayo). *AI Overviews: About last week*. The Keyword — Google Blog. [blog.google/products/search/ai-overviews-update-may-2024](https://blog.google/products/search/ai-overviews-update-may-2024/) ✅ *Verificada 2026-07-28 (fuente primaria). El post oficial sobre el incidente de "comer piedras": de aquí salen el concepto de* data void *y la afirmación de que el fallo no fue de alucinación sino del retriever y del corpus.* — Tomo [[Guia-Maestra-RAG_10-RAG-en-Produccion|10]]
- *Moffatt v. Air Canada*, 2024 BCCRT 149 (Civil Resolution Tribunal of British Columbia, 14 de febrero de 2024), expediente SC-2023-005609. [canlii.org](https://www.canlii.org/en/bc/bccrt/doc/2024/2024bccrt149/2024bccrt149.html) ✅ *Verificada 2026-07-28. Hechos, cifras (812,02 CAD, de los cuales 650,88 en daños) y fundamento (negligent misrepresentation) confirmados por convergencia de fuentes legales. ⚠️ CanLII bloquea el acceso automatizado: **las citas textuales del párrafo 27 conviene contrastarlas en un navegador** antes de reproducirlas fuera de esta guía.* — Tomo [[Guia-Maestra-RAG_10-RAG-en-Produccion|10]]
- Morris, J. X., Kuleshov, V., Shmatikov, V., & Rush, A. M. (2023). "Text Embeddings Reveal (Almost) As Much As Text". *Proceedings of EMNLP 2023*, 12448–12460. DOI 10.18653/v1/2023.emnlp-main.765. arXiv:2310.06816. ✅ *Verificada 2026-07-28 (ACL Anthology; arXiv). **Embedding inversion**; el método se distribuye como* Vec2Text *(nombre del repositorio, no del paper). Recupera exactamente el 92 % de entradas de 32 tokens.* — Tomo [[Guia-Maestra-RAG_10-RAG-en-Produccion|10]]
- Zhuang, S., Koopman, B., Chu, X., & Zuccon, G. (2024). "Understanding and Mitigating the Threat of Vec2Text to Dense Retrieval Systems". *SIGIR-AP '24*, Tokio. arXiv:2402.12784. DOI 10.1145/3673791.3698414. ✅ *Verificada 2026-07-28 (arXiv; venue y DOI vía listado de ACM DL, cuya página bloquea el acceso automatizado). Evalúa las tres defensas que el curso menciona sin citar.* — Tomo [[Guia-Maestra-RAG_10-RAG-en-Produccion|10]]

---

## 12. Quantization y multimodal RAG

- Kusupati, A., Bhatt, G., Rege, A., Wallingford, M., Sinha, A., Ramanujan, V., Howard-Snyder, W., Chen, K., Kakade, S., Jain, P., & Farhadi, A. (2022). "Matryoshka Representation Learning". *Advances in Neural Information Processing Systems 35 (NeurIPS 2022)*. arXiv:2205.13147. ✅ *Verificada 2026-07-28 (proceedings de NeurIPS; arXiv). Nota: la explicación del curso ("dimensiones ordenadas por densidad de información") es una buena intuición divulgativa pero **no es como lo enuncia el paper**, que habla de representaciones* coarse-to-fine *anidadas.* — Tomo [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|11]]
- Faysse, M., Sibille, H., Wu, T., Omrani, B., Viaud, G., Hudelot, C., & Colombo, P. (2025). "ColPali: Efficient Document Retrieval with Vision Language Models". *Proceedings of ICLR 2025*. arXiv:2407.01449. ✅ *Verificada 2026-07-28 (arXiv; proceedings de ICLR). **Es la técnica que el curso llama "PDF RAG"**, nombre que no aparece en la literatura revisada por pares. Introduce el benchmark ViDoRe.* — Tomo [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|11]]
- Macé, Q., Loison, A., & Faysse, M. (2025). *ViDoRe Benchmark V2: Raising the Bar for Visual Retrieval*. arXiv:2505.17166. ✅ *Verificada 2026-07-28. Motivado por la saturación de V1. **Release de benchmark, no paper revisado por pares.*** — Tomo [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|11]]
- Shakir, A., Aarsen, T., & Lee, S. (2024, 22 de marzo). *Binary and Scalar Embedding Quantization for Significantly Faster & Cheaper Retrieval*. Hugging Face Blog. [huggingface.co/blog/embedding-quantization](https://huggingface.co/blog/embedding-quantization) ✅ *Verificada 2026-07-28. Fuente canónica de las cifras de retención de rendimiento (Tom Aarsen mantiene `sentence-transformers`). **Blog técnico de referencia, no paper revisado por pares** — se cita por sus mediciones reproducibles.* — Tomo [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|11]]
- Weaviate (2024, 2 de abril). *32x Reduced Memory Usage With Binary Quantization*. [weaviate.io/blog/binary-quantization](https://weaviate.io/blog/binary-quantization) ✅ *Verificada 2026-07-28. Aporta el contrapunto de recalls ~0,74–0,76 sobre DBPedia.* — Tomo [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|11]]

---

## 13. Modelos, herramientas y documentación oficial

### 13.1 El modelo del curso

> [!warning] ⚠️ La entrada "Meta AI (2024). Llama 3.1" era incitable y se reemplazó
> Nombraba una **familia de modelos**, no un documento: sin autores, sin tipo de obra, sin localizador. La referencia correcta requiere separar tres capas:
>
> - **El paper del modelo:** Grattafiori, A. et al. (2024). *The Llama 3 Herd of Models*. arXiv:2407.21783 (v3). ✅ *Verificada 2026-07-28.* **Dato relevante: el paper cambió de primer autor entre versiones** — la v1 (jul-2024) encabeza con Abhimanyu Dubey y la v3 (nov-2024) con Aaron Grattafiori, de ~559 autores. Por eso circulan "Dubey et al. 2024" y "Grattafiori et al. 2024" y **ambas son legítimas según la versión**; se recomienda citar la v3 e indicar la versión.
> - **Los pesos:** Meta (2024). *Llama-3.1-8B-Instruct* [Model card]. Hugging Face. Lanzamiento: 23 de julio de 2024.
> - **El servicio de inferencia:** el sufijo `-turbo` de `llama-3-1-8b-instruct-turbo` **no es de Meta**: es el nombre del endpoint de Together AI que sirve el modelo cuantizado en FP8.
>
> ✅ **Comprobado (29-jul-2026):** el endpoint fue retirado del catálogo serverless de Together AI el **6 de marzo de 2026** (fuente: `docs.together.ai/docs/deprecations`). Hoy `meta-llama/Meta-Llama-3.1-8B-Instruct-Turbo` solo puede desplegarse como endpoint **dedicado** (bajo demanda o reservado mensual); ya no figura en `together.ai/models` ni en `together.ai/pricing` como opción serverless. Together AI no propuso un reemplazo directo — el propio curso de DeepLearning.AI migró sus ejercicios a `Qwen/Qwen3.5-9B` (reportado en el foro de la comunidad); como alternativa dentro de la familia Llama existe `meta-llama/Llama-3.3-70B-Instruct-Turbo` (70B, sin equivalente 8B serverless activo a la fecha). Quien ejecute el código del curso hoy debe cambiar el modelo antes de correrlo.

### 13.2 Librerías y plataformas

Citadas donde aportan valor. No son "obras" y no llevan ficha bibliográfica.

| Herramienta | Tomos | Qué se usa |
|---|---|---|
| **Weaviate** (cliente Python v4) | 03, 05, 06, 07, 09, 10, 11 | `connect_to_embedded`, `near_text` / `bm25` / `hybrid`, `Filter`, `Rerank`, `MetadataQuery`, `source_properties`, defaults de HNSW |
| **`bm25s`** | 03 | La librería BM25 del assignment C1M2 — *no* `rank_bm25` |
| **`rank_bm25`**, **`scikit-learn`** | 03 | `BM25Okapi`; `TfidfVectorizer` y `CountVectorizer` (variantes de fórmula y defaults) |
| **`sentence-transformers`** | 04, 10, 11 | Sentence embeddings; respalda las cifras de quantization del Tomo 11 |
| **LangChain** | 06 | `RecursiveCharacterTextSplitter`, `SemanticChunker` |
| **OpenTelemetry** | 10 | Estándar de instrumentación del Ungraded Lab 1 |
| **Arize Phoenix** | 10 | Plataforma de observabilidad y evaluación de LLMs |
| **OpenInference** | 10, 11 | Convenciones semánticas de spans (`retrieval.documents.*`, `llm.token_count.*`) |
| **Datadog**, **Grafana** | 10 | Monitoring clásico de infraestructura — lo que Phoenix no cubre |
| **RAGAS** | 09, 10 | Métricas específicas de RAG (ficha bibliográfica en §10) |
| **FAISS**, **Qdrant**, **Pinecone**, **Milvus**, **pgvector**, **Elasticsearch** | 03, 05 | Alternativas comparadas |
| **GLiNER** | 07 | Modelo de NER (ficha en §8) |
| **Pydantic** | 08 | Salida estructurada — el Tomo 08 documenta que el lab genera el schema y **nunca valida** |
| **Together AI**, **OpenAI** | 01, 08, 10, 11 | Endpoints de inferencia OpenAI-compatibles |

### 13.3 Datasets y benchmarks usados

**20 Newsgroups** (11.314 documentos, Tomo 09) · **Pro Git** (Tomo 06, ficha en §7) · dataset de 870 noticias del C1M2 (Tomos 03–04) · **MTEB Retrieval** y **DBPedia** (Tomo 11) · **ViDoRe** (Tomo 11) · **MMLU** y **LLM Arena** (Tomo 08, fichas en §9)

---

## 14. 🔎 La auditoría: qué se corrigió y qué se aprendió

Audiencia: 🔧 🧭

Las 52 obras se verificaron contra fuente primaria el **2026-07-28**, con subagentes que debían devolver **evidencia textual + URL** por cada una, no solo un veredicto.

### 14.1 Los 4 errores corregidos

| Obra | Estaba | Es | Por qué pasó |
|---|---|---|---|
| **Es et al., RAGAS** | 2023, sin venue | **2024**, EACL 2024 System Demonstrations, 150–158 | Se citó el año del preprint y nunca se completó el venue |
| **Liu et al., Lost in the Middle** | TACL, 2023 | TACL, **2024**, 12, 157–173 | Mezcla del año del preprint con el venue de la revista |
| **Johnson et al., FAISS** | 2019 | **2021**, IEEE TBD 7(3), 535–547 | Año del *early access* sellado en el DOI (ver §6) |
| **Meta AI, "Llama 3.1"** | No era una referencia | Paper + model card + endpoint | Nombraba una familia de modelos, no un documento |

### 14.2 Las 6 precisiones

Vaswani et al. → el venue es **NIPS**, no NeurIPS (rebranding en 2018) · LoRA → unificado a **2022/ICLR** · Sentence-BERT → el venue es **EMNLP-IJCNLP** 2019 · Cormack et al. → **Büttcher**, no Buettcher · Mikolov et al. → sin páginas (el workshop track de ICLR 2013 no tuvo actas paginadas) · Aumüller et al. → añadido el número de artículo **101374**.

### 14.3 Dos sospechas que resultaron infundadas

Vale registrarlas, porque **la verificación también sirve para no "corregir" lo que está bien**:

- **Robertson et al., "Okapi at TREC-3" (1995)** — se sospechaba 1996. Es correcto: TREC-3 se celebró en 1994 y sus proceedings salieron en 1995; el 1996 corresponde a **TREC-4**.
- **Malkov & Yashunin, HNSW (2020)** — se sospechaba error de año. Correcto en todos los campos.

### 14.4 El patrón que explica la mayoría de los errores

> [!important] 🎯 Casi todos los errores de esta bibliografía tienen la misma causa
> **Confundir la fecha del preprint, la del *early access* y la del venue formal.** Un mismo trabajo tiene tres fechas legítimas, y los gestores bibliográficos eligen mal:
>
> ```
>    arXiv / preprint  ──►  early access  ──►  issue o proceedings
>    (año A)                (año B, y queda      (año C: EL CITABLE)
>                            sellado en el DOI)
> ```
>
> Afectó a FAISS (2017/2019/**2021**), a RAGAS (2023/**2024**), a Lost in the Middle (2023/**2024**) y a LoRA (2021/**2022**). Y por el mismo mecanismo se sospechó erróneamente de HNSW, cuya cita estaba bien.
>
> **Regla operativa:** cuando el año del DOI no coincida con el año citado, ir a **Crossref o DBLP** y usar la fecha del **issue**. Nunca fiarse de Semantic Scholar para el año: en el caso de FAISS devuelve `year: 2017` junto con el volumen y las páginas del issue de 2021, mezclando dos publicaciones en una ficha.

### 14.5 Tres afirmaciones que se matizaron sin ser errores

- **`k = 60` de RRF** — está en el paper, pero fijado en un piloto y con los autores diciendo que la elección *"no es crítica"*. No es un óptimo demostrado (§5).
- **Las cifras de Contextual Retrieval** — 35 %/49 %/67 % son reducciones de la **tasa de fallo en top-20 chunks** sobre un baseline de 5,7 %, no mejoras de accuracy end-to-end (§7).
- **La atribución del top-k sampling a Fan et al.** — se sostiene, pero los autores lo presentan como decisión metodológica, no como contribución reclamada (§9).

### 14.6 Lo que no se pudo verificar de primera mano

Por transparencia: **ACM Digital Library, IEEE Xplore, SpringerLink, CanLII y OpenReview bloquearon el acceso automatizado** (403 o muros anti-bot). En esos casos la verificación se apoyó en **Crossref, DBLP, ACL Anthology y PDFs alojados por los propios autores** — registros autoritativos, pero no el portal del editor. Está señalado en las fichas afectadas.

Además, dos datos concretos quedaron sin confirmar y **no se dan por buenos**: las páginas de Lewis et al. (2020) y las de ContextCite (NeurIPS no las publica). Se omiten en vez de copiarlas de fuentes secundarias.

---

## 15. Complemento — técnicas avanzadas de query (Tomo 12)

> [!note] Verificadas antes de escribir el tomo (2026-07-28), consolidadas aquí el 2026-09-05
> El Tomo 12 exigía que toda referencia se verificara **antes** de escribir, y así se hizo, con evidencia textual y URL; pero las fichas quedaron solo en su §10.3 y **nunca llegaron a este tomo**. Se consolidan con sus marcas originales (✅ verificada · ⚠️ matiz). Cormack, Clarke & Büttcher (2009) ya figura en §4.

**Antecedente histórico**
- Rocchio, J. J. (1971). "Relevance feedback in information retrieval". En G. Salton (ed.), *The SMART Retrieval System — Experiments in Automatic Document Processing*, 313–323. Prentice Hall. ⚠️ *Verificada vía fuente secundaria autorizada (bibliografía de Manning, Raghavan & Schütze); el documento primario no tiene edición digital pública. Sin DOI.*
- Belkin, N. J., Kantor, P., Fox, E. A., & Shaw, J. A. (1995). "Combining the evidence of multiple query representations for information retrieval". *Information Processing & Management*, 31(3), 431–448. DOI 10.1016/0306-4573(94)00057-A. ✅

**Transformación de queries**
- Ma, X., Gong, Y., He, P., Zhao, H., & Duan, N. (2023). "Query Rewriting for Retrieval-Augmented Large Language Models". *EMNLP 2023*. arXiv:2305.14283. ✅
- Press, O., Zhang, M., Min, S., Schmidt, L., Smith, N. A., & Lewis, M. (2023). "Measuring and Narrowing the Compositionality Gap in Language Models". *Findings of the ACL: EMNLP 2023*, 5687–5711. DOI 10.18653/v1/2023.findings-emnlp.378. ✅ **(Self-Ask — la canónica para RAG)**
- Zhou, D., Schärli, N., Hou, L., Wei, J., Scales, N., Wang, X., Schuurmans, D., Cui, C., Bousquet, O., Le, Q., & Chi, E. (2023). "Least-to-Most Prompting Enables Complex Reasoning in Large Language Models". *ICLR 2023*. arXiv:2205.10625. ✅ ⚠️ *No contiene retrieval — no citar como referencia de RAG.*
- Khot, T., Trivedi, H., Finlayson, M., Fu, Y., Richardson, K., Clark, P., & Sabharwal, A. (2023). "Decomposed Prompting: A Modular Approach for Solving Complex Tasks". *ICLR 2023*. arXiv:2210.02406. ✅
- Zheng, H. S., Mishra, S., Chen, X., Cheng, H.-T., Chi, E. H., Le, Q. V., & Zhou, D. (2024). "Take a Step Back: Evoking Reasoning via Abstraction in Large Language Models". *ICLR 2024*. arXiv:2310.06117. ✅
- Jagerman, R., Zhuang, H., Qin, Z., Wang, X., & Bendersky, M. (2023). *Query Expansion by Prompting Large Language Models*. arXiv:2305.03653. ✅ *Preprint sin venue.*

**Patrones adaptativos**
- Trivedi, H., Balasubramanian, N., Khot, T., & Sabharwal, A. (2023). "Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions". *ACL 2023*, 10014–10037. DOI 10.18653/v1/2023.acl-long.557. ✅ **(IRCoT)**
- Asai, A., Wu, Z., Wang, Y., Sil, A., & Hajishirzi, H. (2024). "Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection". *ICLR 2024 (Oral)*. arXiv:2310.11511. ✅ *El estatus "Oral" está verificado; la afirmación "top 1 %" que circula, no.*
- Yan, S.-Q., Gu, J.-C., Zhu, Y., & Ling, Z.-H. (2024). *Corrective Retrieval Augmented Generation* (CRAG). arXiv:2401.15884. ✅ ⚠️ **Preprint sin venue.**

**Recuperación estructurada**
- Edge, D., Trinh, H., Cheng, N., Bradley, J., Chao, A., Mody, A., Truitt, S., Metropolitansky, D., Ness, R. O., & Larson, J. (2024). *From Local to Global: A Graph RAG Approach to Query-Focused Summarization*. arXiv:2404.16130. ✅ ⚠️ **Preprint sin venue tras más de dos años.** *La v1 lista 8 autores y la v2 añade dos (10); se usa la v2.*
- Sarthi, P., Abdullah, S., Tuli, A., Khanna, S., Goldie, A., & Manning, C. D. (2024). "RAPTOR: Recursive Abstractive Processing for Tree-Organized Retrieval". *ICLR 2024*. arXiv:2401.18059. ✅
- Jiménez Gutiérrez, B., Shu, Y., Gu, Y., Yasunaga, M., & Su, Y. (2024). "HippoRAG: Neurobiologically Inspired Long-Term Memory for Large Language Models". *NeurIPS 2024*. arXiv:2405.14831. ✅
- Jiménez Gutiérrez, B., Shu, Y., Qi, W., Zhou, S., & Su, Y. (2025). "From RAG to Memory: Non-Parametric Continual Learning for Large Language Models" (HippoRAG 2). *ICML 2025*, PMLR 267, 21497–21515. ✅
- Guo, Z., Xia, L., Yu, Y., Ao, T., & Huang, C. (2025). "LightRAG: Simple and Fast Retrieval-Augmented Generation". *Findings of the ACL: EMNLP 2025*, 10746–10761. DOI 10.18653/v1/2025.findings-emnlp.568. ✅ *Enviado a ICLR 2025 y retirado; es **Findings**, no main.*
- Traag, V. A., Waltman, L., & van Eck, N. J. (2019). "From Louvain to Leiden: guaranteeing well-connected communities". *Scientific Reports*, 9, 5233. ⚠️ *Tomada de la cita interna de Edge et al.; confirmar el DOI en la fuente si se cita textualmente.*

**Evaluaciones comparativas**
- Wang, X., Wang, Z., Gao, X., Zhang, F., Wu, Y., Xu, Z., Shi, T., Wang, Z., Li, S., Qian, Q., Yin, R., Lv, C., Zheng, X., & Huang, X. (2024). "Searching for Best Practices in Retrieval-Augmented Generation". *EMNLP 2024*, 17716–17736. arXiv:2407.01219. ✅ **La cita más importante del Tomo 12.**
- Jin, J., Zhu, Y., Dong, G., Zhang, Y., Yang, X., Zhang, C., Zhao, T., Yang, Z., Dou, Z., & Wen, J.-R. (2025). "FlashRAG: A Modular Toolkit for Efficient Retrieval-Augmented Generation Research". *WWW 2025, Resource Track*. arXiv:2405.13576. ✅ ⚠️ *Las cifras citadas proceden de la v1; existe una v2 posterior.*
- Zhang, X., Song, Y., Wang, Y., et al. (2024). "RAGLAB: A Modular and Research-Oriented Unified Framework for Retrieval-Augmented Generation". *EMNLP 2024, System Demonstrations*, 408–418. DOI 10.18653/v1/2024.emnlp-demo.43. ✅
- Rau, D., Déjean, H., Chirkova, N., Formal, T., Wang, S., Nikoulina, V., & Clinchant, S. (2024). "BERGEN: A Benchmarking Library for Retrieval-Augmented Generation". *Findings of EMNLP 2024*, 7640–7663. ✅
- Ammann, P. J. L., Golde, J., & Akbik, A. (2025). *Question Decomposition for Retrieval-Augmented Generation*. *ACL SRW 2025*. arXiv:2507.00355. ✅
- Xiang, Z., Wu, C., Zhang, Q., Chen, S., Hong, Z., Huang, X., & Su, J. (2025). *When to use Graphs in RAG: A Comprehensive Analysis for Graph Retrieval-Augmented Generation*. arXiv:2506.05690. ✅ ⚠️ *Los autores anuncian aceptación en ICLR'26; no confirmada de forma independiente. Tratar como preprint.*
- Han, H., Ma, L., Wang, Y., et al. (2025). *RAG vs. GraphRAG: A Systematic Evaluation and Key Insights*. arXiv:2502.11371. ✅ ⚠️ *Preprint sin venue.*
- Laitenberger, A., Manning, C. D., & Liu, N. F. (2025). "Stronger Baselines for Retrieval-Augmented Generation with Long-Context Language Models". *EMNLP 2025*, 32559–32569. arXiv:2506.03989. ✅
- Hussain, Z., & Nielbo, K. (2026). *The Coverage Illusion: From Pre-retrieval Routing Failure to Post-retrieval Cascades in a Production RAG System*. arXiv:2605.27220. ✅ ⚠️ *Preprint sin venue. Fuente del 27,8 % de consultas que necesitan aumentación.*
- Medrano, L., Verma, A., & Chhabra, M. (2026). *RAG-Fusion en producción*. arXiv:2603.02153. ✅ ⚠️ *Preprint sin venue.*
- Akarsu, M., Karaman, R. K., & Mierbach, C. (2026). *From BM25 to Corrective RAG: Benchmarking Retrieval Strategies for Text-and-Table Documents*. arXiv:2604.01733. ✅ ⚠️ *Preprint sin venue. Fuente de que HyDE empeora en corpus tabulares.*
- Kotte, V. (2026). *Not All Queries Need Rewriting*. arXiv:2603.13301. ✅ ⚠️ *Preprint de autor único.*
- Bigdeli, A., Hamidi Rad, R., Le, H. S., Incesu, M., Arabzadeh, N., Clarke, C. L. A., & Bagheri, E. (2026). *Reproducibility study of LLM query reformulation*. arXiv:2604.27421. ✅ ⚠️ *Preprint sin venue.*
- Eibich, M., Nagpal, S., & Fred-Ojala, A. (2024). *ARAGOG: Advanced RAG Output Grading*. arXiv:2404.01037. ✅ ⚠️ *Preprint sin venue.*
- Ferrazzi, P., Cvjeticanin, M., Piraccini, A., & Giannuzzi, D. (2026). *Is Agentic RAG worth it?*. *ACL 2026 (Industry Track)*. arXiv:2601.07711. ✅
- Iturra-Bocaz, G., & Galuscakova, P. (2026). *A Reproducibility Study of Metacognitive Retrieval-Augmented Generation*. *SIGIR 2026*. arXiv:2604.19899. DOI 10.1145/3805712.3808551. ✅ ⚠️ *La aceptación consta en arXiv y el DOI está registrado; la página de ACM devolvió 403.*

**Fuentes de industria** (citadas como tales, no como literatura académica)
- LangChain (2023, 24 de octubre). *Query Transformations*. Blog post. ✅ *Donde se popularizan multi-query y RAG-Fusion — sin paper fundacional (ver Rackauckas 2024, abajo).*
- Rackauckas, Z. (2024). "RAG-Fusion: a New Take on Retrieval-Augmented Generation". *International Journal on Natural Language Computing (IJNLC)*, 13(1), febrero de 2024. arXiv:2402.03367. ✅ *Verificada 2026-09-05 (arXiv, journal-ref). Existe, pero el §4.1 del Tomo 12 explica por qué NO califica como fundacional: es descriptivo (evalúa "the newly popularized RAG-Fusion method"), el venue es de bajo perfil y la metodología es un case study con evaluación manual.*
- Edge, D., Trinh, H., & Larson, J. (2024, 25 de noviembre). *LazyGraphRAG: Setting a new standard for quality and cost*. Microsoft Research Blog. ✅
- `microsoft/graphrag` (repositorio, licencia MIT). ✅ *README verificado: "not an officially supported Microsoft offering".*

---

## 16. Complemento — frameworks de orquestación (Tomo 13)

> [!note] Verificadas el 28–29 de julio de 2026, antes de escribir; consolidadas aquí el 2026-09-05
> Schluntz & Zhang (2024), *Building effective agents*, ya figura en §10. Lo perecedero del Tomo 13 (versiones, APIs, changelogs) se cita como documentación oficial y **caduca solo**: verificar antes de reutilizar.

**Académicas** — solo DSPy tiene linaje publicado
- Khattab, O., Singhvi, A., Maheshwari, P., Zhang, Z., Santhanam, K., Vardhamanan, S., Haq, S., Sharma, A., Joshi, T. T., Moazam, H., Miller, H., Zaharia, M., & Potts, C. (2024). "DSPy: Compiling Declarative Language Model Calls into State-of-the-Art Pipelines". *ICLR 2024 (Spotlight)*. arXiv:2310.03714. ✅ ⚠️ *El arXiv (v1) dice "Self-Improving Pipelines"; el camera-ready de ICLR dice "State-of-the-Art Pipelines".*
- Khattab, O., Santhanam, K., Li, X. L., Hall, D., Liang, P., Potts, C., & Zaharia, M. (2022). *Demonstrate-Search-Predict*. arXiv:2212.14024. ✅ *Preprint. El predecesor.*
- Opsahl-Ong, K., Ryan, M. J., Purtell, J., Broman, D., Potts, C., Zaharia, M., & Khattab, O. (2024). "Optimizing Instructions and Demonstrations for Multi-Stage Language Model Programs" (MIPRO). *EMNLP 2024*. arXiv:2406.11695. ✅
- Agrawal, L., et al. (2026). "GEPA: Reflective Prompt Evolution Can Outperform Reinforcement Learning". *ICLR 2026 (Oral)*. arXiv:2507.19457. ✅
- Sarmah, B., Dutta, S., Grigoryan, A., Tiwari, M., Pasquali, S., & Mehta, D. (2024). *A Comparative Study of DSPy Teleprompter Algorithms*. arXiv:2412.15298. ✅ *Preprint, muestra pequeña, pero independiente.*
- Aali, A., et al. (2026). *Structured Prompts Improve Evaluation of Language Models*. arXiv:2511.20836. ✅ ⚠️ *No es evaluación independiente: dos coautores son autores del paper de DSPy.*

**Posturas con autoridad**
- Chase, H. (2025, 20 de octubre). *Reflections on Three Years of Building LangChain*. Blog post. ✅
- Husain, H. (2024, 14 de febrero). *"Fuck You, Show Me The Prompt."* Blog post. ✅
- Octomind (2024, ~20 de junio). *Why we no longer use LangChain for building our AI agents*. ⚠️ **`octomind.dev` no resolvía DNS el 2026-07-29**; el contenido y la discusión se conservan en el hilo de Hacker News (id 40739982). *Citar el hilo, no la URL original.*

**Documentación oficial** (perecedera)
- LangChain: changelog (medición de `deepagents` v0.7.0b2), guía de migración a v1, política de versionado y release, tutorial de evaluación de LangSmith, issue de descontinuación de `langchain-community`. ✅ *Verificada julio–agosto de 2026.*
- LlamaIndex, DSPy, Haystack, RAGFlow: repositorios y PyPI. ✅
- OpenTelemetry: convenciones semánticas GenAI (`gen_ai.*`) — **`Status: Development`** a agosto de 2026. ✅

---

## 17. Complemento — structured data RAG (Tomo 14)

> [!note] Verificadas el 2026-09-05, después de escrito el tomo (2026-08-28) — al revés de la regla
> El Tomo 14 se generó sin pasar por el protocolo de verificación y sin consolidar aquí sus fuentes. La auditoría del 2026-09-05 encontró **4 errores** en 7 referencias: un autor inventado ("Baber" por Bahdanau), el título y los autores del paper de DAIL-SQL, el año y venue de BIRD (más una cifra del §1.2 que no provenía del paper) y una URL de documentación muerta. Todos corregidos en el tomo.

- Yu, T., Zhang, R., Yang, K., Yasunaga, M., Wang, D., Li, Z., Ma, J., Li, I., Yao, Q., Roman, S., Zhang, Z., & Radev, D. (2018). "Spider: A Large-Scale Human-Labeled Dataset for Complex and Cross-Domain Semantic Parsing and Text-to-SQL Task". *Proceedings of EMNLP 2018*, 3911–3921. DOI 10.18653/v1/D18-1425. ✅ *Verificada 2026-09-05 (ACL Anthology). 10.181 preguntas y 5.693 consultas SQL únicas sobre 200 bases.*
- Rajkumar, N., Li, R., & Bahdanau, D. (2022). *Evaluating the Text-to-SQL Capabilities of Large Language Models*. arXiv:2204.00498. ✅ *Verificada 2026-09-05 (arXiv; DBLP: preprint sin venue formal). Codex alcanza 67 % de execution accuracy en Spider.*
- Pourreza, M., & Rafiei, D. (2023). "DIN-SQL: Decomposed In-Context Learning of Text-to-SQL with Self-Correction". *Advances in Neural Information Processing Systems 36 (NeurIPS 2023)*, 36339–36348. arXiv:2304.11015. ✅ *Verificada 2026-09-05 (BibTeX oficial de NeurIPS).*
- Gao, D., Wang, H., Li, Y., Sun, X., Qian, Y., Ding, B., & Zhou, J. (2024). "Text-to-SQL Empowered by Large Language Models: A Benchmark Evaluation" (DAIL-SQL). *Proceedings of the VLDB Endowment*, 17(5), 1132–1145. DOI 10.14778/3641204.3641221. arXiv:2308.15363. ✅ *Verificada 2026-09-05 (Crossref; PDF de PVLDB). 86,6 % de EX en Spider.*
- Li, J., Hui, B., Qu, G., Yang, J., Li, B., Li, B., Wang, B., Qin, B., Geng, R., Huo, N., Zhou, X., Ma, C., Li, G., Chang, K., Huang, F., Cheng, R., & Li, Y. (2023). "Can LLM Already Serve as A Database Interface? A BIg Bench for Large-Scale Database Grounded Text-to-SQLs" (BIRD). *Advances in Neural Information Processing Systems 36 (NeurIPS 2023), Datasets and Benchmarks Track*, 42330–42357. arXiv:2305.03111. ✅ *Verificada 2026-09-05 (proceedings de NeurIPS). GPT-4: 54,89 % de EX frente a 92,96 % humano.*
- LangChain (s.f.). *Build a SQL agent*. Documentación oficial: docs.langchain.com/oss/python/langchain/sql-agent ✅ *Verificada 2026-09-05. La URL que citaba el tomo (`docs/use_cases/sql`) devuelve 404.*
- LlamaIndex (s.f.). *NL SQL table — `NLSQLTableQueryEngine`*. Referencia de API: developers.llamaindex.ai/python/framework-api-reference/query_engine/NL_SQL_table/ ✅ *Verificada 2026-09-05 (la clase se exporta desde `llama_index.core.query_engine`; `docs.llamaindex.ai` redirige al dominio nuevo).*

---

## 18. Adiciones de la revisión del 2026-08-28 (Tomos 04, 06, 09, 10)

> [!note] Verificadas el 2026-09-05
> La revisión del 2026-08-28 añadió contenido a los Tomos 04 (Matryoshka, quantization, modelos), 06 (late chunking), 09 (LLM-as-judge) y 10 (guardrails) sin pasar sus fuentes por verificación ni por este tomo. Resultado de la auditoría: **una referencia inexistente** ("Yamada et al. (2024), *Scalar and Binary Quantization for ANN Search*" — fusión de un post de Hugging Face con un paper de 2021), **un título incorrecto** (Günther et al.) y nueve fichas correctas o con precisiones menores. Kusupati et al. (2022), Shakir, Aarsen & Lee (2024) y Anthropic (2024, *Contextual Retrieval*) ya figuraban en §12 y §7.

- Yamada, I., Asai, A., & Hajishirzi, H. (2021). "Efficient Passage Retrieval with Hashing for Open-domain Question Answering" (BPR). *Proceedings of ACL-IJCNLP 2021, Volume 2: Short Papers*, 979–986. DOI 10.18653/v1/2021.acl-short.123. ✅ *Verificada 2026-09-05 (ACL Anthology). El paper que el post de Hugging Face cita como origen del paso de rescore.* — Tomo 04
- Nussbaum, Z., Morris, J. X., Duderstadt, B., & Mulyar, A. (2025). "Nomic Embed: Training a Reproducible Long Context Text Embedder". *Transactions on Machine Learning Research*. arXiv:2402.01613. ✅ *Verificada 2026-09-05 (arXiv "Accepted to TMLR"; DBLP). Describe v1.* — Tomo 04
- Nomic AI (2024, 14 de febrero). *Unboxing Nomic Embed v1.5: Resizable Production Embeddings with Matryoshka Representation Learning*. Blog post y model card `nomic-ai/nomic-embed-text-v1.5`. ✅ *Verificada 2026-09-05. Fuente del soporte Matryoshka (64–768 dimensiones).* — Tomo 04
- Sturua, S., Mohr, I., Akram, M. K., Günther, M., Wang, B., Krimmel, M., Wang, F., Mastrapas, G., Koukounas, A., Wang, N., & Xiao, H. (2024). *jina-embeddings-v3: Multilingual Embeddings With Task LoRA*. arXiv:2409.10173. Versión revisada por pares con otro título: "Jina Embeddings V3: Multilingual Text Encoder with Low-Rank Adaptations", *ECIR 2025*, LNCS, 123–129, DOI 10.1007/978-3-031-88720-8_21. ✅ *Verificada 2026-09-05 (arXiv; Crossref).* — Tomo 04
- OpenAI (2024, 25 de enero). *New embedding models and API updates*. Anuncio oficial; y *Create embeddings* (referencia de API, parámetro `dimensions`, "only supported in text-embedding-3 and later models"). ✅ *Verificada 2026-09-05 (documentación oficial; openai.com devolvió 403 al fetcher, la fecha se confirmó por registros del mismo día).* — Tomo 04
- Koenig, D., & Shakir, A. (2024, 12 de abril). *64 bytes per embedding, yee-haw*. Mixedbread Blog; y model card `mixedbread-ai/mxbai-embed-large-v1`. ✅ *Verificada 2026-09-05. El post de lanzamiento (Lee, Shakir, Koenig & Lipp, 8 de marzo de 2024) aún decía que la versión Matryoshka estaba "in the making"; el soporte MRL + binario lo documenta el post de abril.* — Tomo 04
- Günther, M., Mohr, I., Williams, D. J., Wang, B., & Xiao, H. (2024). *Late Chunking: Contextual Chunk Embeddings Using Long-Context Embedding Models*. arXiv:2409.04701 (v3, 2025). ✅ *Verificada 2026-09-05 (arXiv; DBLP: preprint). Título corregido ("Embeddings", no "Representations").* — Tomo 06
- Liu, Y., Iter, D., Xu, Y., Wang, S., Xu, R., & Zhu, C. (2023). "G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment". *Proceedings of EMNLP 2023*, 2511–2522. DOI 10.18653/v1/2023.emnlp-main.153. arXiv:2303.16634. ✅ *Verificada 2026-09-05 (ACL Anthology).* — Tomo 09
- Zheng, L., Chiang, W.-L., Sheng, Y., Zhuang, S., Wu, Z., Zhuang, Y., Lin, Z., Li, Z., Li, D., Xing, E. P., Zhang, H., Gonzalez, J. E., & Stoica, I. (2023). "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena". *Advances in Neural Information Processing Systems 36 (NeurIPS 2023), Datasets and Benchmarks Track*, 46595–46623. arXiv:2306.05685. ✅ *Verificada 2026-09-05 (BibTeX oficial de NeurIPS). Distinta del paper de Chatbot Arena de 2024 (§9).* — Tomo 09
- Rebedea, T., Dinu, R., Sreedhar, M. N., Parisien, C., & Cohen, J. (2023). "NeMo Guardrails: A Toolkit for Controllable and Safe LLM Applications with Programmable Rails". *Proceedings of EMNLP 2023: System Demonstrations*, 431–445. DOI 10.18653/v1/2023.emnlp-demo.40. ✅ *Verificada 2026-09-05 (ACL Anthology).* — Tomo 10
- Inan, H., Upasani, K., Chi, J., Rungta, R., Iyer, K., Mao, Y., Tontchev, M., Hu, Q., Fuller, B., Testuggine, D., & Khabsa, M. (2023). *Llama Guard: LLM-based Input-Output Safeguard for Human-AI Conversations*. arXiv:2312.06674. ✅ *Verificada 2026-09-05 (arXiv; DBLP: preprint sin venue formal).* — Tomo 10

---

## 🔗 Conexiones

- [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|Tomo 15 · Glosario ejecutivo]] — el vocabulario que estas fuentes sostienen.
- [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC]] — índice y tracker de la guía.
- [[Instrucciones|📖 Instrucciones]] — el §5 fija la política de fuentes que este tomo materializa.

> [!note] Cómo se mantiene este tomo
> Cada tomo nuevo suma sus referencias aquí, **verificadas antes de escribirse**, no después. Si una obra no se puede verificar contra fuente primaria, se cita igual pero **marcada como tal** — nunca se presenta como confirmada. El §14 crece con cada auditoría: los errores encontrados son parte del valor de la guía, no una vergüenza que ocultar.

---

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|Tomo 15 · Glosario ejecutivo]]
