---
title: "Tomo 16 — Bibliografía"
tags: [rag, bibliografia, referencias, transversal, verificacion]
audiencias: [tecnico, puente, ejecutivo]
tomo: 16
version: 1.0
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
> Todas las obras citadas en los Tomos 01–11, con sus datos de publicación completos. Las citas en el texto usan el formato `(Autor, Año)`.
>
> ✅ **52/52 obras verificadas contra fuente primaria (2026-07-28)** — cada una con evidencia y DOI/URL. La auditoría corrigió **4 errores** y precisó **6 fichas** más; el detalle está en la sección 14, porque los errores encontrados son didácticos en sí mismos.
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

## 🔗 Conexiones

- [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|Tomo 15 · Glosario ejecutivo]] — el vocabulario que estas fuentes sostienen.
- [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC]] — índice y tracker de la guía.
- [[Instrucciones|📖 Instrucciones]] — el §5 fija la política de fuentes que este tomo materializa.

> [!note] Cómo se mantiene este tomo
> Cada tomo nuevo suma sus referencias aquí, **verificadas antes de escribirse**, no después. Si una obra no se puede verificar contra fuente primaria, se cita igual pero **marcada como tal** — nunca se presenta como confirmada. El §14 crece con cada auditoría: los errores encontrados son parte del valor de la guía, no una vergüenza que ocultar.

---

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|Tomo 15 · Glosario ejecutivo]]
