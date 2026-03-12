[youtub chatbot Backup 3 (8).json](https://github.com/user-attachments/files/25546331/youtub.chatbot.Backup.3.8.json)<img width="1110" height="389" alt="image" src="https://github.com/user-attachments/assets/b4571b03-c68f-465c-9872-ce5b87b61544" />
[Uploading youtub {
  "name": "youtub chatbot Backup 3",
  "nodes": [
    {
      "parameters": {
        "public": true,
        "mode": "webhook",
        "options": {
          "allowedOrigins": "*",
          "responseMode": "responseNode"
        }
      },
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.1,
      "position": [
        -1600,
        -96
      ],
      "id": "59f959fc-0ffa-4c69-bdb8-390f2b5e3be6",
      "name": "When chat message received",
      "webhookId": "b72e5ddf-d17c-45c1-b68d-85d731ad4a6b"
    },
    {
      "parameters": {
        "text": "={{ $json.chatInput }}{{ $json.body[0].chatInput }}",
        "translateTo": "=EN-US",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.deepL",
      "typeVersion": 1,
      "position": [
        -1712,
        -976
      ],
      "id": "f51ee760-2bf5-4fdb-b3e2-52885f7b1dec",
      "name": "Translate a language",
      "credentials": {
        "deepLApi": {
          "id": "a9rJcw5hVBATnLJS",
          "name": "DeepL account 2"
        }
      }
    },
    {
      "parameters": {
        "text": "={{ $json.faq_answer_en }}",
        "translateTo": "={{ $('Translate a language').item.json.detected_source_language }}",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.deepL",
      "typeVersion": 1,
      "position": [
        -736,
        -960
      ],
      "id": "a3f71614-49c1-42e9-b4bf-c963859ecae9",
      "name": "Translate a language1",
      "credentials": {
        "deepLApi": {
          "id": "a9rJcw5hVBATnLJS",
          "name": "DeepL account 2"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "// Build FAQ Index (multilingual guardrails for 45 Magrid FAQs)\n// Place this Code node right before your AI Chat/Agent classifier.\n\nconst data = $getWorkflowStaticData('global');\nconst faqs = Array.isArray(data.faq) ? data.faq : [];\n\n// 1) Read translated question (EN) and detected language from upstream translate node\nconst qRaw = String($json.user_question_en ?? $json.text ?? $json.chatInput ?? \"\").trim();\nconst userLang = String($json.detected_source_language ?? \"\").trim(); // e.g., \"fa\", \"es\", \"pt\", \"fr\", \"de\", ...\n\n// Fallback: if no question, pass through (classifier should return none)\nif (!qRaw) {\n  return [{\n    user_question: \"\",\n    faq_index: faqs.map(f => ({ id: f.id, q: f.q })).slice(0, 6), // tiny list so LLM doesn't choke\n    detected_source_language: userLang || \"en\",\n  }];\n}\n\nconst q = qRaw.toLowerCase();\n\n// 2) Multilingual MAP: keywords/synonyms per FAQ\n// Tip: keep each list short but distinctive; include EN + FA + ES + PT + FR + DE\nconst MAP = [\n  { id:\"faq-001\", kw:[\"what is magrid\",\"about magrid\",\"who is it for\",\"برای کی\",\"براى چه کسانى\",\"para quién\",\"quem é\",\"pour qui\",\"für wen\",\"was ist magrid\"] },\n  { id:\"faq-002\", kw:[\"different\",\"unique\",\"how different\",\"vs other\",\"فرقش\",\"تفاوت\",\"diferente\",\"único\",\"différent\",\"anders\",\"vergleich\",\"vs\"] },\n  { id:\"faq-003\", kw:[\"skills\",\"problem solving\",\"memory\",\"spatial\",\"logical\",\"مهارت\",\"حل مسئله\",\"حافظه\",\"فضایی\",\"lógico\",\"habilidades\",\"memoria\",\"espacial\",\"compétences\",\"logique\",\"fähigkeiten\",\"räumlich\"] },\n  { id:\"faq-004\", kw:[\"research\",\"studies\",\"neuroscience\",\"evidence\",\"scientific\",\"تحقیق\",\"مطالعه\",\"عصب\",\"investigación\",\"pesquisa\",\"recherche\",\"forschung\",\"wissenschaftlich\"] },\n  { id:\"faq-005\", kw:[\"assessment\",\"pre-test\",\"mid-test\",\"post-test\",\"track progress\",\"ارزیابی\",\"پیش آزمون\",\"میان آزمون\",\"پس آزمون\",\"evaluación\",\"avaliação\",\"évaluation\",\"bewertung\",\"tests\"] },\n  { id:\"faq-006\", kw:[\"ages\",\"age\",\"grade\",\"grades\",\"4 to 8\",\"سن\",\"مقطع\",\"edades\",\"idades\",\"âge\",\"alter\",\"klassen\"] },\n  { id:\"faq-007\", kw:[\"inclusive\",\"accessibility\",\"language-free\",\"包容\",\"فراگیر\",\"دسترسی\",\"inclusivo\",\"accesible\",\"inclusif\",\"barrierefrei\",\"ohne sprache\"] },\n  { id:\"faq-008\", kw:[\"multilingual\",\"multi-lingual\",\"many languages classroom\",\"چندزبانه\",\"aulas multilingues\",\"aula multilingue\",\"clase multilingüe\",\"classe multilingue\",\"mehrsprachige klasse\"] },\n  { id:\"faq-009\", kw:[\"special educational needs\",\"sen\",\"learning disabilities\",\"developmental delays\",\"cognitive challenges\",\"نیازهای ویژه\",\"ناتوانی\",\"dificultades de aprendizaje\",\"necessidades especiais\",\"besoins éducatifs particuliers\",\"lernbehinderungen\"] },\n  { id:\"faq-010\", kw:[\"autism\",\"adhd\",\"sensory\",\"overstimulating\",\"no rewards\",\"اوتیسم\",\"بیش‌فعالی\",\"حسی\",\"autismo\",\"tdah\",\"sensorial\",\"autisme\",\"adhs\",\"reizüberflutung\"] },\n  { id:\"faq-011\", kw:[\"hearing\",\"deaf\",\"hearing impairment\",\"audio optional\",\"کم‌شنوایی\",\"ناشنوا\",\"audición\",\"auditivo\",\"surdité\",\"hörgeschädigt\"] },\n  { id:\"faq-012\", kw:[\"implement\",\"onboarding\",\"start in school\",\"deployment\",\"استقرار\",\"پیاده‌سازی\",\"implementar\",\"implantação\",\"déployer\",\"einführen\"] },\n  { id:\"faq-013\", kw:[\"curriculum\",\"common core\",\"ib\",\"alignment\",\"standards\",\"برنامه درسی\",\"استاندارد\",\"currículo\",\"alinhamento\",\"programme\",\"richtlinien\"] },\n  { id:\"faq-014\", kw:[\"any time\",\"mid-year\",\"during school year\",\"flexible start\",\"هر زمان\",\"وسط سال\",\"a mitad de año\",\"em qualquer altura\",\"à tout moment\",\"jederzeit\"] },\n  { id:\"faq-015\", kw:[\"how long\",\"duration\",\"complete\",\"finish\",\"self-paced\",\"چقدر طول میکشد\",\"مدت\",\"duración\",\"duração\",\"durée\",\"dauer\",\"selbstgesteuert\"] },\n  { id:\"faq-016\", kw:[\"how often\",\"usage\",\"screen time\",\"1 hour per week\",\"15-20 minutes\",\"هر چند وقت\",\"مدت استفاده\",\"tiempo de pantalla\",\"tempo de ecrã\",\"temps d’écran\",\"bildschirmzeit\"] },\n  { id:\"faq-017\", kw:[\"teacher training\",\"training required\",\"onboarding sessions\",\"quick-start\",\"آموزش معلم\",\"capacitación\",\"formação\",\"formation\",\"lehrerschulung\"] },\n  { id:\"faq-018\", kw:[\"monitor progress\",\"teacher dashboard\",\"real-time reports\",\"analytics\",\"گزارش پیشرفت\",\"داشبورد\",\"informes\",\"relatórios\",\"rapports\",\"berichte\"] },\n  { id:\"faq-019\", kw:[\"early detection\",\"dyslexia\",\"dysgraphia\",\"dyscalculia\",\"pre-screening\",\"تشخیص زودهنگام\",\"dislexia\",\"disgrafía\",\"discalculia\",\"dépistage\",\"früherkennung\"] },\n  { id:\"faq-020\", kw:[\"early intervention\",\"remedial\",\"remediation\",\"مداخله زودهنگام\",\"intervención temprana\",\"intervenção precoce\",\"intervention précoce\",\"frühförderung\"] },\n  { id:\"faq-021\", kw:[\"institutions\",\"who can use\",\"schools\",\"ngos\",\"community centers\",\"چه نهادهایی\",\"instituciones\",\"instituições\",\"institutions\",\"einrichtungen\"] },\n  { id:\"faq-022\", kw:[\"responsible technology\",\"who recommendations\",\"screen-time limits\",\"healthy digital habits\",\"مصرف مسئولانه\",\"hábitos saludables\",\"usages responsables\",\"verantwortungsvoll\"] },\n  { id:\"faq-023\", kw:[\"home use\",\"homeschool\",\"at home\",\"family\",\"استفاده در خانه\",\"uso en casa\",\"uso em casa\",\"à la maison\",\"zuhause\"] },\n  { id:\"faq-024\", kw:[\"parent access\",\"download app\",\"app store\",\"google play\",\"microsoft store\",\"دسترسی والدین\",\"acceso padres\",\"acesso pais\",\"accès parent\",\"elternzugang\"] },\n  { id:\"faq-025\", kw:[\"multiple children\",\"same device\",\"profiles\",\"چند کودک\",\"چند پروفایل\",\"varios niños\",\"vários filhos\",\"plusieurs enfants\",\"mehrere kinder\"] },\n  { id:\"faq-026\", kw:[\"safe\",\"gdpr\",\"coppa\",\"no ads\",\"no external links\",\"ایمن\",\"تبلیغ ندارد\",\"seguro\",\"sécurité\",\"sicher\",\"keine werbung\"] },\n  { id:\"faq-027\", kw:[\"parent progress\",\"parent reports\",\"parental insights\",\"گزارش والدین\",\"informes a padres\",\"relatórios aos pais\",\"rapports parents\",\"elternberichte\"] },\n  { id:\"faq-028\", kw:[\"licenses\",\"types of licenses\",\"subscriptions\",\"مجوز\",\"planos\",\"suscripciones\",\"licences\",\"lizenzen\"] },\n  { id:\"faq-029\", kw:[\"trial\",\"free demo\",\"try before purchasing\",\"دمو\",\"آزمایشی\",\"prueba\",\"teste\",\"essai\",\"demo\"] },\n  { id:\"faq-030\", kw:[\"upgrade\",\"change license\",\"scalable\",\"flexible\",\"ارتقا\",\"cambiar plan\",\"actualizar\",\"mise à niveau\",\"upgrade\",\"wechseln\"] },\n  { id:\"faq-031\", kw:[\"user accounts\",\"profiles managed\",\"teacher admin parent\",\"حساب کاربری\",\"cuentas\",\"contas\",\"comptes\",\"konten\"] },\n  { id:\"faq-032\", kw:[\"data protection\",\"privacy\",\"gdpr compliance\",\"terms\",\"حریم خصوصی\",\"protección de datos\",\"proteção de dados\",\"protection des données\",\"datenschutz\"] },\n  { id:\"faq-033\", kw:[\"price\",\"tarif\",\"tarifs\",\"pricing\",\"cost\",\"how much\",\"subscription cost\",\"قیمت\",\"هزینه\",\"precio\",\"rate\",\"preço\",\"prix\",\"preis\",\"kostet\"] },\n  { id:\"faq-034\", kw:[\"devices\",\"android\",\"ios\",\"tablet\",\"computer\",\"touch screen\",\"دستگاه\",\"dispositivos\",\"appareil\",\"geräte\"] },\n  { id:\"faq-035\", kw:[\"internet\",\"offline\",\"online\",\"sync\",\"connect weekly\",\"آفلاین\",\"بدون اینترنت\",\"sin internet\",\"sem internet\",\"hors-ligne\",\"offline\"] },\n  { id:\"faq-036\", kw:[\"language\",\"languages\",\"spanish\",\"french\",\"german\",\"portuguese\",\"luxembourgish\",\"nepali\",\"english\",\"زبان\",\"idiomas\",\"línguas\",\"langues\",\"sprachen\"] },\n  { id:\"faq-037\", kw:[\"support\",\"technical support\",\"help\",\"whatsapp\",\"email\",\"پشتیبانی\",\"suporte\",\"soporte\",\"assistance\",\"support technique\",\"hilfe\"] },\n  { id:\"faq-038\", kw:[\"how quickly\",\"implemented in a day\",\"set up quickly\",\"سریع راه‌اندازی\",\"rápido\",\"rapidement\",\"schnell einrichten\"] },\n  { id:\"faq-039\", kw:[\"training and support for educators\",\"materials\",\"continued support\",\"آموزش و پشتیبانی معلمان\",\"formación y soporte\",\"formação e apoio\",\"formation et soutien\",\"schulung und support\"] },\n  { id:\"faq-040\", kw:[\"rural\",\"urban\",\"low-resource\",\"different classroom contexts\",\"سازگار\",\"کم‌منبع\",\"rural/urbano\",\"recursos limitados\",\"milieux à faibles ressources\",\"ländlich/städtisch\"] },\n  { id:\"faq-041\", kw:[\"complement\",\"other math programs\",\"compatible\",\"تکمیل\",\"complementar\",\"complementario\",\"complément\",\"ergänzen\"] },\n  { id:\"faq-042\", kw:[\"monitoring at scale\",\"district\",\"ngo level\",\"dashboards\",\"aggregated data\",\"پایش سراسری\",\"seguimiento a escala\",\"monitorização em escala\",\"pilotage à l’échelle\",\"monitoring in großem maßstab\"] },\n  { id:\"faq-043\", kw:[\"reporting outcomes\",\"stakeholders\",\"export reports\",\"funding partners\",\"گزارش به ذی‌نفعان\",\"informes a financiadores\",\"relatórios a parceiros\",\"rapports aux parties prenantes\",\"berichte für stakeholder\"] },\n  { id:\"faq-044\", kw:[\"where implemented\",\"countries\",\"which countries\",\"global presence\",\"کشورها\",\"países\",\"pays\",\"länder\",\"wo implementiert\"] },\n  { id:\"faq-045\", kw:[\"implemented in other countries\",\"partnerships\",\"ministries\",\"expand\",\"گسترش\",\"شراکت\",\"otros países\",\"outros países\",\"autres pays\",\"andere länder\",\"partnerschaften\"] },\n  { id:\"faq-046\", kw:[\"one-time pay\",\"once\",\"without abonment\",\"no subscription\",\"دفعة واحدة\", \"مرة واحدة\", \"بدون اشتراك\", \"بدون اشتراك\",\"pago único\", \"una sola vez\", \"sin abono\", \"sin suscripción\",\"paiement unique\", \"une seule fois\", \"sans abonnement\" , \"Einmalzahlung\", \"einmalig\", \"ohne Abo\", \"kein Abonnement\",\"eemol Bezuelung\", \"eemol\", \"ouni Ofbezuelung\", \"keen Abonnement\",\"pagamento único\", \"uma vez\", \"sem subscrição\", \"sem mensalidade\"] },\n];\n\n// 3) Guardrail: find direct keyword hits\nconst hits = [];\nfor (const m of MAP) {\n  if (m.kw.some(k => q.includes(k))) hits.push(m.id);\n}\n\n// 4) Shortlist:\n// - if guardrail hit(s): shortlist those IDs\n// - else: use lightweight token overlap to pick top-N (default 12)\nlet shortlist = [];\nif (hits.length) {\n  const set = new Set(hits);\n  shortlist = faqs.filter(f => set.has(f.id)).map(f => ({ id: f.id, q: f.q }));\n} else {\n  const toks = q.split(/\\s+/).filter(t => t && t.length > 2);\n  function score(text) {\n    const t = text.toLowerCase();\n    let s = 0;\n    for (const tok of toks) if (t.includes(tok)) s += 1;\n    return s;\n  }\n  const scored = faqs.map(f => ({ id: f.id, q: f.q, _s: score(f.q) }));\n  scored.sort((a,b)=>b._s - a._s);\n  const N = Math.min(12, scored.length || 12);\n  shortlist = scored.slice(0, N).map(({id,q}) => ({ id, q }));\n}\n\n// Safety: never send empty list to the LLM (send a tiny subset)\nif (!shortlist.length) {\n  shortlist = faqs.slice(0, 6).map(f => ({ id: f.id, q: f.q }));\n}\n\nreturn [{\n  user_question: qRaw,                    // English question\n  faq_index: shortlist,                   // Shortlist for the classifier\n  detected_source_language: userLang || \"en\",\n  faqs: data.faq,\n  faqs_count: faqs.length\n}];\n"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        -800,
        -320
      ],
      "id": "e3a05a27-1693-4355-9cd4-71b9d6aee94f",
      "name": "Code1"
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=User question:\n={{ $json.user_question }}\n\nFAQ list (JSON):\n={{ JSON.stringify($json.faq_index) }}\n",
        "options": {
          "systemMessage": "You are a strict FAQ classifier for Magrid.\n\nINPUTS:\n- user_question: a short, possibly noisy user query\n- faq_index: a JSON array of {id, q}, already shortlisted\n\nYOUR TASK:\n1) Select the SINGLE best matching FAQ id from faq_index.\n2) If none is clearly relevant, return id \"none\".\n3) For extremely short queries (≤2 tokens), require a direct keyword/synonym overlap with one of the FAQ questions; otherwise return \"none\".\n4) Be conservative. Do not guess based on generic words.\n5) Return ONLY valid JSON:\n   {\"id\":\"<faq-id-or-none>\", \"reason\":\"<brief>\", \"confidence\": <0..1>}\n\nEXAMPLES:\nUser: \"language?\" \n→ {\"id\":\"faq-036\",\"reason\":\"Asks about supported languages; faq-036 covers language availability.\",\"confidence\":0.95}\n\nUser: \"what languages do you have in the app?\"\n→ {\"id\":\"faq-036\",\"reason\":\"Directly about supported languages.\",\"confidence\":0.95}\n\nUser: \"multilingual classroom?\"\n→ {\"id\":\"faq-008\",\"reason\":\"Asks about multilingual environments; faq-008 addresses this.\",\"confidence\":0.9}\n\nUser: \"pricing / cost\"\n→ {\"id\":\"faq-033\",\"reason\":\"Asks about cost; faq-033 covers pricing.\",\"confidence\":0.9}\n\nUser: \"devices?\"\n→ {\"id\":\"faq-034\",\"reason\":\"Asks about supported devices.\",\"confidence\":0.95}\n\nUser: \"offline / internet needed?\"\n→ {\"id\":\"faq-035\",\"reason\":\"Connectivity/offline usage; faq-035.\",\"confidence\":0.95}\n\nUser: \"ages / grades?\"\n→ {\"id\":\"faq-006\",\"reason\":\"Supported ages/grades.\",\"confidence\":0.95}\n\nUser: \"return policy\" \n→ {\"id\":\"none\",\"reason\":\"Not in Magrid FAQ list (store-like topic).\",\"confidence\":0.0}\n\nUser: \"?\"\n→ {\"id\":\"none\",\"reason\":\"No meaningful query.\",\"confidence\":0.0}\n"
        }
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 2.2,
      "position": [
        -464,
        -320
      ],
      "id": "b02f25f6-fd5c-4173-a4bc-9cc4eb0d2370",
      "name": "AI Agent"
    },
    {
      "parameters": {
        "model": {
          "__rl": true,
          "value": "gpt-5",
          "mode": "list",
          "cachedResultName": "gpt-5"
        },
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.2,
      "position": [
        -464,
        -112
      ],
      "id": "255781b5-c0cc-47fb-85c6-7a24ba1122ec",
      "name": "OpenAI Chat Model",
      "credentials": {
        "openAiApi": {
          "id": "WYx43EFCfNuobSYP",
          "name": "OpenAi account 2"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "// Code2 (FAQ_Init)\n// Save all FAQs into workflow static data\n\nconst data = $getWorkflowStaticData('global');\n\nif (!Array.isArray(data.faq) || data.faq.length === 0) {\n  data.faq = [\n    { id: \"faq-001\", q: \"What is Magrid and who is it for?\", a: \"Magrid is an inclusive, language-free digital platform that teaches early mathematics and cognitive skills to children from 4 years old. It is ideal for schools, families, special education programs and NGOs with social impact projects.\" },\n    { id: \"faq-002\", q: \"How is Magrid different from other programs?\", a: \"Unlike traditional platforms, Magrid uses visual instructions and requires no reading or spoken language. It is grounded in neuroscience and designed to support children with diverse learning needs.\" },\n    { id: \"faq-003\", q: \"What skills does Magrid develop?\", a: \"Magrid helps children improve problem-solving, memory, spatial awareness, and logical thinking—without needing to read.\" },\n    { id: \"faq-004\", q: \"Is Magrid based on scientific research?\", a: \"Yes. Magrid is backed by over 10 years of research in neuroscience, child development, and psychology. It is supported by six published studies, with more ongoing, led by the University of Luxembourg in collaboration with universities in Germany, France, and Portugal.\" },\n    { id: \"faq-005\", q: \"Does Magrid include assessments to track student progress?\", a: \"Yes. Magrid features a continuous assessment system with three key assessments: a pre-test, a mid-test, and a post-test. These track progress and support data-driven instruction.\" },\n    { id: \"faq-006\", q: \"What age or grade levels does Magrid support?\", a: \"Magrid is designed for Early Childhood Education and early Primary School (approximately ages 4 to 8) and beyond this age for children with special educational needs and different developmental needs.\" },\n\n    { id: \"faq-007\", q: \"Why is Magrid considered inclusive?\", a: \"Magrid’s language-free design ensures accessibility for all learners, including those with learning difficulties, special needs, or limited literacy.\" },\n    { id: \"faq-008\", q: \"Can Magrid be used in multilingual environments?\", a: \"Yes, Magrid is ideal for multilingual classrooms as it relies on visual instructions, making it accessible regardless of the child's spoken language.\" },\n    { id: \"faq-009\", q: \"Does Magrid support children with special educational needs?\", a: \"Yes, Magrid is designed to adapt to a wide range of learners, including children with developmental delays, cognitive challenges, and other learning disabilities.\" },\n    { id: \"faq-010\", q: \"Is Magrid suitable for children with autism or ADHD?\", a: \"Yes. Magrid is sensory-friendly and intentionally designed without overstimulating elements or gamified reward systems. It fosters positive learning through gentle reinforcement, allows children to progress at their own pace, and avoids penalizing mistakes—reducing frustration and supporting self-confidence.\" },\n    { id: \"faq-011\", q: \"Can children with hearing impairments use Magrid?\", a: \"Absolutely. Magrid is mostly visual and doesn’t rely on spoken instructions, making it effective for children with hearing loss. A small number of activities include optional audio, but these can be skipped without affecting progress.\" },\n\n    { id: \"faq-012\", q: \"How can Magrid be implemented in schools or learning centers?\", a: \"Magrid can be used on tablets or touch screen devices. We provide training, onboarding sessions, and ongoing support to help educators integrate Magrid into their teaching.\" },\n    { id: \"faq-013\", q: \"Does Magrid align with existing curricula?\", a: \"Yes. Magrid builds key foundational skills that fit into any curriculum. It is already mapped to Common Core, IB, and several national standards, with curriculum alignment guides available.\" },\n    { id: \"faq-014\", q: \"Can I implement Magrid at any time during the school year?\", a: \"Absolutely. Magrid can be integrated at any point in the academic year, making it a flexible support tool to reinforce math concepts and cognitive development.\" },\n    { id: \"faq-015\", q: \"How long does the Magrid program take to complete?\", a: \"There is no fixed duration. Magrid is self-paced and adapts to each child’s learning speed, allowing them to progress based on their individual development.\" },\n    { id: \"faq-016\", q: \"How often should students use Magrid?\", a: \"We recommend 1 hour of use per week, delivered in 15–20 minute sessions, 3 to 4 times per week. The screen time can be adjusted in the app’s settings.\" },\n    { id: \"faq-017\", q: \"Is teacher training required?\", a: \"Training is not required but strongly recommended. We provide free onboarding, quick-start guides, and live sessions for teachers.\" },\n    { id: \"faq-018\", q: \"How is student progress monitored?\", a: \"Teachers and administrators have access to real-time progress reports showing learning paths, strengths, and areas for improvement. Basic analytics are available within the app, while detailed class- and school-level reports are provided through the teacher/admin website panel.\" },\n    { id: \"faq-019\", q: \"How does Magrid support early detection of learning difficulties?\", a: \"Through its assessment system and detailed progress reports, Magrid helps identify early signs of challenges such as dyslexia, dysgraphia, and dyscalculia, making it a valuable pre-screening tool.\" },\n    { id: \"faq-020\", q: \"Can Magrid be used in early intervention or remedial programs?\", a: \"Absolutely. Its step-by-step progression and adaptive design make Magrid especially useful for early intervention and remedial instruction.\" },\n    { id: \"faq-021\", q: \"Which institutions can use Magrid?\", a: \"Magrid is used in schools, preschools, NGOs, inclusive education programs, and community centers.\" },\n    { id: \"faq-022\", q: \"How does Magrid promote responsible technology use?\", a: \"Magrid is designed to align with the World Health Organization’s recommendations on screen time for young children. We limit use to short, focused sessions — no more than 1 hour per week — to ensure healthy digital habits and balanced development.\" },\n\n    { id: \"faq-023\", q: \"Can I use Magrid at home?\", a: \"Yes. Magrid is suitable for home use and homeschooling. It provides an independent learning journey supported by parental insights.\" },\n    { id: \"faq-024\", q: \"How do I get access as a parent?\", a: \"Parents can download the Magrid App from the different App stores: Apple Store, Google Play and Microsoft Store. We offer licenses for home use. Contact us via our website if you need access.\" },\n    { id: \"faq-025\", q: \"Can multiple children use the same device?\", a: \"Yes, you can create multiple child profiles on a single device.\" },\n    { id: \"faq-026\", q: \"Is Magrid safe for children to use alone?\", a: \"Absolutely. The app has no ads, no external links, and complies with child safety standards (GDPR, COPPA).\" },\n    { id: \"faq-027\", q: \"How can I track my child’s progress?\", a: \"Parents receive regular progress updates and can view detailed reports via the app.\" },\n\n    { id: \"faq-028\", q: \"What types of licenses are available?\", a: \"We offer licenses for schools, NGOs, institutions, and families. Licenses can be customized based on the number of students and duration. Check our different subscriptions here.\" },\n    { id: \"faq-029\", q: \"Can I try Magrid before purchasing?\", a: \"Yes. We offer free demos and trial access.\" },\n    { id: \"faq-030\", q: \"Can I upgrade or change my license?\", a: \"Yes, licenses are flexible and scalable at any point during the subscription.\" },\n    { id: \"faq-031\", q: \"How are user accounts managed?\", a: \"Student profiles can be created and managed by teachers, administrators, or parents.\" },\n    { id: \"faq-032\", q: \"Does Magrid comply with data protection regulations?\", a: \"Yes. Magrid strictly adheres to GDPR and other international data protection laws, as outlined in our Terms & Conditions.\" },\n    { id: \"faq-033\", q: \"How much does Magrid cost?\", a: \"Parents: Pricing is region-specific and adapted to local exchange rates within each app store. Flexible plans are available (3 or 12 months), and each subscription covers all children within a family account. Educators: single teacher account for small groups, or tailored solutions for larger classes. Schools/organizations: institutional packages (from ~15 students) with full teacher training, 24/7 support, offline resources, and impact reports. you can also register and check in your account(https://panel.magrid.education/Account/Welcome) or chat with our support in whatsapp and ask: +352 661 337704\" },\n\n    { id: \"faq-034\", q: \"On which devices can Magrid be used?\", a: \"Magrid works on Android and iOS (Apple) tablets, and computers. Tablets and digital devices with touch screens are recommended.\" },\n    { id: \"faq-035\", q: \"Does Magrid require internet access?\", a: \"An internet connection is only needed to download the app and receive updates. Students can use Magrid offline for all learning activities. Schools need to connect the devices to the internet regularly (at least once a week) to sync their students’ progress.\" },\n    { id: \"faq-036\", q: \"Is Magrid available in multiple languages?\", a: \"Yes. Magrid is currently available in English, Spanish, French, German, Portuguese, Luxembourgish and Nepali.\" },\n    { id: \"faq-037\", q: \"Is technical support available?\", a: \"Yes. We offer Whatsapp and email support 365 days.\" },\n\n    { id: \"faq-038\", q: \"How quickly can Magrid be implemented in a classroom or center?\", a: \"Magrid can be set up and used within a single day. With quick-start guides and optional free onboarding sessions, educators can start working with their students right away.\" },\n    { id: \"faq-039\", q: \"What kind of training and support is available for educators?\", a: \"We offer live onboarding sessions, training materials, and continued support via email and Whatsapp. Our team is available throughout the implementation to ensure a smooth experience.\" },\n    { id: \"faq-040\", q: \"How does Magrid adapt to different classroom contexts (rural, urban, low-resource)?\", a: \"Magrid is flexible and scalable. Its offline capabilities and visual design make it effective in both low-resource settings and well-equipped classrooms.\" },\n    { id: \"faq-041\", q: \"Can Magrid be used to complement other math programs?\", a: \"Yes. Magrid is compatible with most early math curricula and is often used as a complementary tool to strengthen foundational math and cognitive skills.\" },\n    { id: \"faq-042\", q: \"How does Magrid support monitoring at scale (e.g., district or NGO level)?\", a: \"Administrators can access dashboards with aggregated data across schools or regions, allowing for easy tracking of impact, usage, and student progress.\" },\n    { id: \"faq-043\", q: \"Does Magrid offer any tools for reporting outcomes to stakeholders?\", a: \"Yes. We provide progress reports that can be exported and shared with parents, school leadership, or funding partners.\" },\n\n    { id: \"faq-044\", q: \"Where is Magrid currently implemented?\", a: \"Magrid is being used in schools and communities across 10+ countries, including Luxembourg, Portugal, Nepal, Peru, Brazil, Colombia, Curaçao, Ecuador, India, Ukraine, Laos... and others joining in 2025.\" },\n    { id: \"faq-045\", q: \"Can Magrid be implemented in other countries?\", a: \"Yes. We partner with local institutions, schools, NGOs, and ministries of education to bring Magrid to new communities. Contact us to explore opportunities: contact@magrid.education.\" },\n    { id: \"faq-048\", q: \"What are the rates for families?\" , a:\"Parents: Pricing is region-specific and adapted to local exchange rates within each app store. Flexible plans are available (3 or 12 months), and each subscription covers all children within a family account. Educators: single teacher account for small groups, or tailored solutions for larger classes. Schools/organizations: institutional packages (from ~15 students) with full teacher training, 24/7 support, offline resources, and impact reports. you can also register and check in your account(https://panel.magrid.education/Account/Welcome) or chat with our support in whatsapp and ask: +352 661 337704\"},\n    { id: \"faq-046\", q: \"possibilities to buy the access once, without an abo?\", a: \"Yes, a one-time payment option is available. Please contact contact@magrid.education, and they will assist you with the process.\" }\n  ];\n}\n\n// مهم: ورودی قبلی (chatInput) رو هم پاس بدیم\nreturn [{ ...$json, faqs: data.faq, ok: true, count: data.faq.length }];\n"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        -1072,
        -320
      ],
      "id": "11f00409-13e5-4cd2-9e22-67b24c27c22c",
      "name": "Code4"
    },
    {
      "parameters": {
        "text": "={{ $json.faq_answer_en }}",
        "translateTo": "={{ $('Translate a language2').item.json.detected_source_language }}",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.deepL",
      "typeVersion": 1,
      "position": [
        -1424,
        -960
      ],
      "id": "5b330d81-9444-448c-822a-c6533c43c3b0",
      "name": "Translate a language3",
      "credentials": {
        "deepLApi": {
          "id": "a9rJcw5hVBATnLJS",
          "name": "DeepL account 2"
        }
      }
    },
    {
      "parameters": {
        "authentication": "webhook",
        "content": "={{ $json.chatInput }} {{ $json.output }} {{ $json.text }}",
        "options": {}
      },
      "type": "n8n-nodes-base.discord",
      "typeVersion": 2,
      "position": [
        -2608,
        -1024
      ],
      "id": "36ed01d7-0e33-4cf2-8dd2-437671a12deb",
      "name": "Discord2",
      "webhookId": "0eb5e71a-5272-4a29-8912-2ab78da05d7b",
      "credentials": {
        "discordWebhookApi": {
          "id": "LpkTAbVqO0s6RZFK",
          "name": "Discord Webhook account 2"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "let parsed = {};\ntry {\n  parsed = JSON.parse($json.output || \"{}\");\n} catch(e) {\n  parsed = {};\n}\n\nconst id = parsed.id || \"none\";\n\nconst faqMap = {\n  \"faq-001\": \"Magrid is an inclusive, language-free digital platform that teaches early mathematics and cognitive skills to children from 4 years old. It is ideal for schools, families, special education programs and NGOs with social impact projects.\",\n  \"faq-002\": \"Unlike traditional platforms, Magrid uses visual instructions and requires no reading or spoken language. It is grounded in neuroscience and designed to support children with diverse learning needs.\",\n  \"faq-003\": \"Magrid helps children improve problem-solving, memory, spatial awareness, and logical thinking—without needing to read.\",\n  \"faq-004\": \"Magrid is backed by over 10 years of research in neuroscience, child development, and psychology. It is supported by six published studies, with more ongoing, led by the University of Luxembourg in collaboration with universities in Germany, France, and Portugal.\",\n  \"faq-005\": \"Magrid features a continuous assessment system with three key assessments: a pre-test, a mid-test, and a post-test. These track progress and support data-driven instruction.\",\n  \"faq-006\": \"Magrid is designed for Early Childhood Education and early Primary School (approximately ages 4 to 8) and beyond this age for children with special educational needs and different developmental needs.\",\n  \"faq-007\": \"Magrid’s language-free design ensures accessibility for all learners, including those with learning difficulties, special needs, or limited literacy.\",\n  \"faq-008\": \"Magrid is ideal for multilingual classrooms as it relies on visual instructions, making it accessible regardless of the child's spoken language.\",\n  \"faq-009\": \"Magrid is designed to adapt to a wide range of learners, including children with developmental delays, cognitive challenges, and other learning disabilities.\",\n  \"faq-010\": \"Magrid is sensory-friendly and intentionally designed without overstimulating elements or gamified reward systems. It fosters positive learning through gentle reinforcement, allows children to progress at their own pace, and avoids penalizing mistakes—reducing frustration and supporting self-confidence.\",\n  \"faq-011\": \"Absolutely. Magrid is mostly visual and doesn’t rely on spoken instructions, making it effective for children with hearing loss. A small number of activities include optional audio, but these can be skipped without affecting progress.\",\n  \"faq-012\": \"Magrid can be used on tablets or touch screen devices. We provide training, onboarding sessions, and ongoing support to help educators integrate Magrid into their teaching.\",\n  \"faq-013\": \"Magrid builds key foundational skills that fit into any curriculum. It is already mapped to Common Core, IB, and several national standards, with curriculum alignment guides available.\",\n  \"faq-014\": \"Absolutely. Magrid can be integrated at any point in the academic year, making it a flexible support tool to reinforce math concepts and cognitive development.\",\n  \"faq-015\": \"There is no fixed duration. Magrid is self-paced and adapts to each child’s learning speed, allowing them to progress based on their individual development.\",\n  \"faq-016\": \"We recommend 1 hour of use per week, delivered in 15–20 minute sessions, 3 to 4 times per week. The screen time can be adjusted in the app’s settings.\",\n  \"faq-017\": \"Training is not required but strongly recommended. We provide free onboarding, quick-start guides, and live sessions for teachers.\",\n  \"faq-018\": \"Teachers and administrators have access to real-time progress reports showing learning paths, strengths, and areas for improvement. Basic analytics are available within the app, while detailed class- and school-level reports are provided through the teacher/admin website panel.\",\n  \"faq-019\": \"Through its assessment system and detailed progress reports, Magrid helps identify early signs of challenges such as dyslexia, dysgraphia, and dyscalculia, making it a valuable pre-screening tool.\",\n  \"faq-020\": \"Absolutely. Its step-by-step progression and adaptive design make Magrid especially useful for early intervention and remedial instruction.\",\n  \"faq-021\": \"Magrid is used in schools, preschools, NGOs, inclusive education programs, and community centers.\",\n  \"faq-022\": \"Magrid is designed to align with the World Health Organization’s recommendations on screen time for young children. We limit use to short, focused sessions — no more than 1 hour per week — to ensure healthy digital habits and balanced development.\",\n  \"faq-023\": \"Magrid is suitable for home use and homeschooling. It provides an independent learning journey supported by parental insights.\",\n  \"faq-024\": \"Parents can download the Magrid App from the different App stores: Apple Store, Google Play and Microsoft Store. We offer licenses for home use. Contact us via our website if you need access.\",\n  \"faq-025\": \"you can create multiple child profiles on a single device.\",\n  \"faq-026\": \"Absolutely. The app has no ads, no external links, and complies with child safety standards (GDPR, COPPA).\",\n  \"faq-027\": \"Parents receive regular progress updates and can view detailed reports via the app.\",\n  \"faq-028\": \"We offer licenses for schools, NGOs, institutions, and families. Licenses can be customized based on the number of students and duration. Check our different subscriptions here.\",\n  \"faq-029\": \"We offer free demos and trial access.\",\n  \"faq-030\": \"licenses are flexible and scalable at any point during the subscription.\",\n  \"faq-031\": \"Student profiles can be created and managed by teachers, administrators, or parents.\",\n  \"faq-032\": \"Magrid strictly adheres to GDPR and other international data protection laws, as outlined in our Terms & Conditions.\",\n  \"faq-033\": \"Parents: Pricing is region-specific and adapted to local exchange rates within each app store. Flexible plans are available (3 or 12 months), and each subscription covers all children within a family account. Educators: single teacher account for small groups, or tailored solutions for larger classes. Schools/organizations: institutional packages (from ~15 students) with full teacher training, 24/7 support, offline resources, and impact reports. you can also register and check in your account(https://panel.magrid.education/Account/Welcome) or chat with our support in whatsapp and ask: +352 661 337704\",\n  \"faq-034\": \"Magrid works on Android and iOS (Apple) tablets, and computers. Tablets and digital devices with touch screens are recommended.\",\n  \"faq-035\": \"An internet connection is only needed to download the app and receive updates. Students can use Magrid offline for all learning activities. Schools need to connect the devices to the internet regularly (at least once a week) to sync their students’ progress.\",\n  \"faq-036\": \"Magrid is currently available in English, Spanish, French, German, Portuguese, Luxembourgish and Nepali.\",\n  \"faq-037\": \"We offer WhatsApp and email support 365 days a year. you can also send WhatsApp message to +352661337704\",\n  \"faq-038\": \"Magrid can be set up and used within a single day. With quick-start guides and optional free onboarding sessions, educators can start working with their students right away.\",\n  \"faq-039\": \"We offer live onboarding sessions, training materials, and continued support via email and WhatsApp. Our team is available throughout the implementation to ensure a smooth experience.\",\n  \"faq-040\": \"Magrid is flexible and scalable. Its offline capabilities and visual design make it effective in both low-resource settings and well-equipped classrooms.\",\n  \"faq-041\": \"Magrid is compatible with most early math curricula and is often used as a complementary tool to strengthen foundational math and cognitive skills.\",\n  \"faq-042\": \"Administrators can access dashboards with aggregated data across schools or regions, allowing for easy tracking of impact, usage, and student progress.\",\n  \"faq-043\": \"We provide progress reports that can be exported and shared with parents, school leadership, or funding partners.\",\n  \"faq-044\": \"Magrid is being used in schools and communities across 10+ countries, including: Luxembourg, Portugal, Nepal, Peru, Brazil, Colombia, Curaçao, Ecuador, India, Ukraine, Laos... and others joining in 2025.\",\n  \"faq-046\": \"Hello\",\n  \"faq-045\": \"We partner with local institutions, schools, NGOs, and ministries of education to bring Magrid to new communities. Contact us to explore opportunities.\",\n  \"faq-046\": \"Yes, a one-time payment option is available. Please contact contact@magrid.education, and they will assist you with the process.\",\n  \"faq-047\": \"Hello, How can I help you? 😊\"\n};\n\nlet answer = \"🫤 Oops—No answer available at the moment. Share your Email 📧 or contact us on 💬 WhatsApp +352661337704 and we’ll reply soon ✌🏻!\";\nif (faqMap[id]) {\n  answer = faqMap[id];\n}\n\nreturn [{\n  ...$json,\n  faq_answer_en: answer,\n}];\n"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        -64,
        -320
      ],
      "id": "b0f90942-ac8a-484e-a29b-da1d434fb14b",
      "name": "Code5"
    },
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "c0001645-7703-4590-8179-4de420c492d3",
        "responseMode": "responseNode",
        "options": {}
      },
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2.1,
      "position": [
        -1584,
        -544
      ],
      "id": "67a16335-8218-4e7e-82c1-e4c181b254e4",
      "name": "Webhook1",
      "webhookId": "c0001645-7703-4590-8179-4de420c492d3"
    },
    {
      "parameters": {},
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.3,
      "position": [
        -368,
        -112
      ],
      "id": "33599718-536a-434f-bd13-b4bea6966490",
      "name": "Simple Memory1"
    },
    {
      "parameters": {
        "mode": "chooseBranch"
      },
      "type": "n8n-nodes-base.merge",
      "typeVersion": 3.2,
      "position": [
        176,
        -304
      ],
      "id": "4b0f34bf-9b85-4ff9-8787-240c4e326eb9",
      "name": "Merge2"
    },
    {
      "parameters": {
        "authentication": "webhook",
        "content": "={{ $('When chat message received').item.json.chatInput }}{{ \n\" ==> Question  \" + ($json.chatInput ?? \"\") + \n\"\\n\\nAnswer: \" + ($json.faq_answer_en ?? \"\") \n}}\n",
        "options": {}
      },
      "type": "n8n-nodes-base.discord",
      "typeVersion": 2,
      "position": [
        208,
        32
      ],
      "id": "baf58138-ecb0-409e-b71e-4360f8f0da73",
      "name": "Discord3",
      "webhookId": "0eb5e71a-5272-4a29-8912-2ab78da05d7b",
      "credentials": {
        "discordWebhookApi": {
          "id": "q9CB6zSWBEuVHGJt",
          "name": "Discord Webhook account 5"
        }
      }
    },
    {
      "parameters": {
        "mode": "chooseBranch",
        "useDataOfInput": "={{ 1 }}"
      },
      "type": "n8n-nodes-base.merge",
      "typeVersion": 3.2,
      "position": [
        -432,
        -976
      ],
      "id": "dc10ec96-39c8-45a9-86e6-541b235532a3",
      "name": "Merge3"
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api-free.deepl.com/v2/translate",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "DeepL-Auth-Key 4aa052d6-3dd6-47ed-a705-2008eed0a004:fx"
            }
          ]
        },
        "sendBody": true,
        "contentType": "form-urlencoded",
        "bodyParameters": {
          "parameters": [
            {
              "name": "text",
              "value": "={{$json.chatInput}}"
            },
            {
              "name": "target_lang",
              "value": "EN"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.4,
      "position": [
        -1136,
        -960
      ],
      "id": "b4c3fdea-a091-4c2f-ae54-aefc107b4e26",
      "name": "HTTP Request2"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "0c6c60b6-eddb-4db9-802f-b415db204a1f",
              "name": "respond",
              "value": "={{ $('Code5').item.json.faq_answer_en }}",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        720,
        -48
      ],
      "id": "a4acf780-f043-4bdd-a2ea-db1627488285",
      "name": "Edit Fields"
    },
    {
      "parameters": {
        "respondWith": "text",
        "responseBody": "={{ $json.faq_answer_en }}",
        "options": {}
      },
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1.5,
      "position": [
        480,
        -304
      ],
      "id": "5e0a6576-9983-448d-a8a3-69bd887278f1",
      "name": "Respond to Webhook"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.noOp",
      "typeVersion": 1,
      "position": [
        432,
        32
      ],
      "id": "4059e14a-478c-4391-9ad5-45558ef6ddcc",
      "name": "No Operation, do nothing"
    }
  ],
  "pinData": {},
  "connections": {
    "When chat message received": {
      "main": [
        [
          {
            "node": "Code4",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Translate a language": {
      "main": [
        []
      ]
    },
    "Translate a language1": {
      "main": [
        []
      ]
    },
    "Code1": {
      "main": [
        [
          {
            "node": "AI Agent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "AI Agent": {
      "main": [
        [
          {
            "node": "Code5",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "OpenAI Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "AI Agent",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Code4": {
      "main": [
        [
          {
            "node": "Code1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Translate a language3": {
      "main": [
        []
      ]
    },
    "Discord2": {
      "main": [
        []
      ]
    },
    "Code5": {
      "main": [
        [
          {
            "node": "Merge2",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Webhook1": {
      "main": [
        [
          {
            "node": "Code4",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Simple Memory1": {
      "ai_memory": [
        [
          {
            "node": "AI Agent",
            "type": "ai_memory",
            "index": 0
          }
        ]
      ]
    },
    "Merge2": {
      "main": [
        [
          {
            "node": "Respond to Webhook",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Discord3": {
      "main": [
        [
          {
            "node": "No Operation, do nothing",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "HTTP Request2": {
      "main": [
        []
      ]
    },
    "Merge3": {
      "main": [
        []
      ]
    },
    "Edit Fields": {
      "main": [
        []
      ]
    },
    "Respond to Webhook": {
      "main": [
        [
          {
            "node": "Discord3",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": true,
  "settings": {
    "executionOrder": "v1"
  },
  "versionId": "9229e0f2-1992-4361-82e2-7acc1c814136",
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "e277aaeb25adaa67f5f8cb9dae56e3aeabf19f993fd064fc3ed2591aca3f5042"
  },
  "id": "vaNhsX2BjFnEQbLj",
  "tags": []
}chatbot Backup 3 (8).json…]()

--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------

{
  "name": "youtub chatbot Backup 3",
  "nodes": [
    {
      "parameters": {
        "public": true,
        "mode": "webhook",
        "options": {
          "allowedOrigins": "*",
          "responseMode": "responseNode"
        }
      },
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.1,
      "position": [
        -1600,
        -96
      ],
      "id": "59f959fc-0ffa-4c69-bdb8-390f2b5e3be6",
      "name": "When chat message received",
      "webhookId": "b72e5ddf-d17c-45c1-b68d-85d731ad4a6b"
    },
    {
      "parameters": {
        "text": "={{ $json.chatInput }}{{ $json.body[0].chatInput }}",
        "translateTo": "=EN-US",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.deepL",
      "typeVersion": 1,
      "position": [
        -1712,
        -976
      ],
      "id": "f51ee760-2bf5-4fdb-b3e2-52885f7b1dec",
      "name": "Translate a language",
      "credentials": {
        "deepLApi": {
          "id": "a9rJcw5hVBATnLJS",
          "name": "DeepL account 2"
        }
      }
    },
    {
      "parameters": {
        "text": "={{ $json.faq_answer_en }}",
        "translateTo": "={{ $('Translate a language').item.json.detected_source_language }}",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.deepL",
      "typeVersion": 1,
      "position": [
        -736,
        -960
      ],
      "id": "a3f71614-49c1-42e9-b4bf-c963859ecae9",
      "name": "Translate a language1",
      "credentials": {
        "deepLApi": {
          "id": "a9rJcw5hVBATnLJS",
          "name": "DeepL account 2"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "// Build FAQ Index (multilingual guardrails for 45 Magrid FAQs)\n// Place this Code node right before your AI Chat/Agent classifier.\n\nconst data = $getWorkflowStaticData('global');\nconst faqs = Array.isArray(data.faq) ? data.faq : [];\n\n// 1) Read translated question (EN) and detected language from upstream translate node\nconst qRaw = String($json.user_question_en ?? $json.text ?? $json.chatInput ?? \"\").trim();\nconst userLang = String($json.detected_source_language ?? \"\").trim(); // e.g., \"fa\", \"es\", \"pt\", \"fr\", \"de\", ...\n\n// Fallback: if no question, pass through (classifier should return none)\nif (!qRaw) {\n  return [{\n    user_question: \"\",\n    faq_index: faqs.map(f => ({ id: f.id, q: f.q })).slice(0, 6), // tiny list so LLM doesn't choke\n    detected_source_language: userLang || \"en\",\n  }];\n}\n\nconst q = qRaw.toLowerCase();\n\n// 2) Multilingual MAP: keywords/synonyms per FAQ\n// Tip: keep each list short but distinctive; include EN + FA + ES + PT + FR + DE\nconst MAP = [\n  { id:\"faq-001\", kw:[\"what is magrid\",\"about magrid\",\"who is it for\",\"برای کی\",\"براى چه کسانى\",\"para quién\",\"quem é\",\"pour qui\",\"für wen\",\"was ist magrid\"] },\n  { id:\"faq-002\", kw:[\"different\",\"unique\",\"how different\",\"vs other\",\"فرقش\",\"تفاوت\",\"diferente\",\"único\",\"différent\",\"anders\",\"vergleich\",\"vs\"] },\n  { id:\"faq-003\", kw:[\"skills\",\"problem solving\",\"memory\",\"spatial\",\"logical\",\"مهارت\",\"حل مسئله\",\"حافظه\",\"فضایی\",\"lógico\",\"habilidades\",\"memoria\",\"espacial\",\"compétences\",\"logique\",\"fähigkeiten\",\"räumlich\"] },\n  { id:\"faq-004\", kw:[\"research\",\"studies\",\"neuroscience\",\"evidence\",\"scientific\",\"تحقیق\",\"مطالعه\",\"عصب\",\"investigación\",\"pesquisa\",\"recherche\",\"forschung\",\"wissenschaftlich\"] },\n  { id:\"faq-005\", kw:[\"assessment\",\"pre-test\",\"mid-test\",\"post-test\",\"track progress\",\"ارزیابی\",\"پیش آزمون\",\"میان آزمون\",\"پس آزمون\",\"evaluación\",\"avaliação\",\"évaluation\",\"bewertung\",\"tests\"] },\n  { id:\"faq-006\", kw:[\"ages\",\"age\",\"grade\",\"grades\",\"4 to 8\",\"سن\",\"مقطع\",\"edades\",\"idades\",\"âge\",\"alter\",\"klassen\"] },\n  { id:\"faq-007\", kw:[\"inclusive\",\"accessibility\",\"language-free\",\"包容\",\"فراگیر\",\"دسترسی\",\"inclusivo\",\"accesible\",\"inclusif\",\"barrierefrei\",\"ohne sprache\"] },\n  { id:\"faq-008\", kw:[\"multilingual\",\"multi-lingual\",\"many languages classroom\",\"چندزبانه\",\"aulas multilingues\",\"aula multilingue\",\"clase multilingüe\",\"classe multilingue\",\"mehrsprachige klasse\"] },\n  { id:\"faq-009\", kw:[\"special educational needs\",\"sen\",\"learning disabilities\",\"developmental delays\",\"cognitive challenges\",\"نیازهای ویژه\",\"ناتوانی\",\"dificultades de aprendizaje\",\"necessidades especiais\",\"besoins éducatifs particuliers\",\"lernbehinderungen\"] },\n  { id:\"faq-010\", kw:[\"autism\",\"adhd\",\"sensory\",\"overstimulating\",\"no rewards\",\"اوتیسم\",\"بیش‌فعالی\",\"حسی\",\"autismo\",\"tdah\",\"sensorial\",\"autisme\",\"adhs\",\"reizüberflutung\"] },\n  { id:\"faq-011\", kw:[\"hearing\",\"deaf\",\"hearing impairment\",\"audio optional\",\"کم‌شنوایی\",\"ناشنوا\",\"audición\",\"auditivo\",\"surdité\",\"hörgeschädigt\"] },\n  { id:\"faq-012\", kw:[\"implement\",\"onboarding\",\"start in school\",\"deployment\",\"استقرار\",\"پیاده‌سازی\",\"implementar\",\"implantação\",\"déployer\",\"einführen\"] },\n  { id:\"faq-013\", kw:[\"curriculum\",\"common core\",\"ib\",\"alignment\",\"standards\",\"برنامه درسی\",\"استاندارد\",\"currículo\",\"alinhamento\",\"programme\",\"richtlinien\"] },\n  { id:\"faq-014\", kw:[\"any time\",\"mid-year\",\"during school year\",\"flexible start\",\"هر زمان\",\"وسط سال\",\"a mitad de año\",\"em qualquer altura\",\"à tout moment\",\"jederzeit\"] },\n  { id:\"faq-015\", kw:[\"how long\",\"duration\",\"complete\",\"finish\",\"self-paced\",\"چقدر طول میکشد\",\"مدت\",\"duración\",\"duração\",\"durée\",\"dauer\",\"selbstgesteuert\"] },\n  { id:\"faq-016\", kw:[\"how often\",\"usage\",\"screen time\",\"1 hour per week\",\"15-20 minutes\",\"هر چند وقت\",\"مدت استفاده\",\"tiempo de pantalla\",\"tempo de ecrã\",\"temps d’écran\",\"bildschirmzeit\"] },\n  { id:\"faq-017\", kw:[\"teacher training\",\"training required\",\"onboarding sessions\",\"quick-start\",\"آموزش معلم\",\"capacitación\",\"formação\",\"formation\",\"lehrerschulung\"] },\n  { id:\"faq-018\", kw:[\"monitor progress\",\"teacher dashboard\",\"real-time reports\",\"analytics\",\"گزارش پیشرفت\",\"داشبورد\",\"informes\",\"relatórios\",\"rapports\",\"berichte\"] },\n  { id:\"faq-019\", kw:[\"early detection\",\"dyslexia\",\"dysgraphia\",\"dyscalculia\",\"pre-screening\",\"تشخیص زودهنگام\",\"dislexia\",\"disgrafía\",\"discalculia\",\"dépistage\",\"früherkennung\"] },\n  { id:\"faq-020\", kw:[\"early intervention\",\"remedial\",\"remediation\",\"مداخله زودهنگام\",\"intervención temprana\",\"intervenção precoce\",\"intervention précoce\",\"frühförderung\"] },\n  { id:\"faq-021\", kw:[\"institutions\",\"who can use\",\"schools\",\"ngos\",\"community centers\",\"چه نهادهایی\",\"instituciones\",\"instituições\",\"institutions\",\"einrichtungen\"] },\n  { id:\"faq-022\", kw:[\"responsible technology\",\"who recommendations\",\"screen-time limits\",\"healthy digital habits\",\"مصرف مسئولانه\",\"hábitos saludables\",\"usages responsables\",\"verantwortungsvoll\"] },\n  { id:\"faq-023\", kw:[\"home use\",\"homeschool\",\"at home\",\"family\",\"استفاده در خانه\",\"uso en casa\",\"uso em casa\",\"à la maison\",\"zuhause\"] },\n  { id:\"faq-024\", kw:[\"parent access\",\"download app\",\"app store\",\"google play\",\"microsoft store\",\"دسترسی والدین\",\"acceso padres\",\"acesso pais\",\"accès parent\",\"elternzugang\"] },\n  { id:\"faq-025\", kw:[\"multiple children\",\"same device\",\"profiles\",\"چند کودک\",\"چند پروفایل\",\"varios niños\",\"vários filhos\",\"plusieurs enfants\",\"mehrere kinder\"] },\n  { id:\"faq-026\", kw:[\"safe\",\"gdpr\",\"coppa\",\"no ads\",\"no external links\",\"ایمن\",\"تبلیغ ندارد\",\"seguro\",\"sécurité\",\"sicher\",\"keine werbung\"] },\n  { id:\"faq-027\", kw:[\"parent progress\",\"parent reports\",\"parental insights\",\"گزارش والدین\",\"informes a padres\",\"relatórios aos pais\",\"rapports parents\",\"elternberichte\"] },\n  { id:\"faq-028\", kw:[\"licenses\",\"types of licenses\",\"subscriptions\",\"مجوز\",\"planos\",\"suscripciones\",\"licences\",\"lizenzen\"] },\n  { id:\"faq-029\", kw:[\"trial\",\"free demo\",\"try before purchasing\",\"دمو\",\"آزمایشی\",\"prueba\",\"teste\",\"essai\",\"demo\"] },\n  { id:\"faq-030\", kw:[\"upgrade\",\"change license\",\"scalable\",\"flexible\",\"ارتقا\",\"cambiar plan\",\"actualizar\",\"mise à niveau\",\"upgrade\",\"wechseln\"] },\n  { id:\"faq-031\", kw:[\"user accounts\",\"profiles managed\",\"teacher admin parent\",\"حساب کاربری\",\"cuentas\",\"contas\",\"comptes\",\"konten\"] },\n  { id:\"faq-032\", kw:[\"data protection\",\"privacy\",\"gdpr compliance\",\"terms\",\"حریم خصوصی\",\"protección de datos\",\"proteção de dados\",\"protection des données\",\"datenschutz\"] },\n  { id:\"faq-033\", kw:[\"price\",\"tarif\",\"tarifs\",\"pricing\",\"cost\",\"how much\",\"subscription cost\",\"قیمت\",\"هزینه\",\"precio\",\"rate\",\"preço\",\"prix\",\"preis\",\"kostet\"] },\n  { id:\"faq-034\", kw:[\"devices\",\"android\",\"ios\",\"tablet\",\"computer\",\"touch screen\",\"دستگاه\",\"dispositivos\",\"appareil\",\"geräte\"] },\n  { id:\"faq-035\", kw:[\"internet\",\"offline\",\"online\",\"sync\",\"connect weekly\",\"آفلاین\",\"بدون اینترنت\",\"sin internet\",\"sem internet\",\"hors-ligne\",\"offline\"] },\n  { id:\"faq-036\", kw:[\"language\",\"languages\",\"spanish\",\"french\",\"german\",\"portuguese\",\"luxembourgish\",\"nepali\",\"english\",\"زبان\",\"idiomas\",\"línguas\",\"langues\",\"sprachen\"] },\n  { id:\"faq-037\", kw:[\"support\",\"technical support\",\"help\",\"whatsapp\",\"email\",\"پشتیبانی\",\"suporte\",\"soporte\",\"assistance\",\"support technique\",\"hilfe\"] },\n  { id:\"faq-038\", kw:[\"how quickly\",\"implemented in a day\",\"set up quickly\",\"سریع راه‌اندازی\",\"rápido\",\"rapidement\",\"schnell einrichten\"] },\n  { id:\"faq-039\", kw:[\"training and support for educators\",\"materials\",\"continued support\",\"آموزش و پشتیبانی معلمان\",\"formación y soporte\",\"formação e apoio\",\"formation et soutien\",\"schulung und support\"] },\n  { id:\"faq-040\", kw:[\"rural\",\"urban\",\"low-resource\",\"different classroom contexts\",\"سازگار\",\"کم‌منبع\",\"rural/urbano\",\"recursos limitados\",\"milieux à faibles ressources\",\"ländlich/städtisch\"] },\n  { id:\"faq-041\", kw:[\"complement\",\"other math programs\",\"compatible\",\"تکمیل\",\"complementar\",\"complementario\",\"complément\",\"ergänzen\"] },\n  { id:\"faq-042\", kw:[\"monitoring at scale\",\"district\",\"ngo level\",\"dashboards\",\"aggregated data\",\"پایش سراسری\",\"seguimiento a escala\",\"monitorização em escala\",\"pilotage à l’échelle\",\"monitoring in großem maßstab\"] },\n  { id:\"faq-043\", kw:[\"reporting outcomes\",\"stakeholders\",\"export reports\",\"funding partners\",\"گزارش به ذی‌نفعان\",\"informes a financiadores\",\"relatórios a parceiros\",\"rapports aux parties prenantes\",\"berichte für stakeholder\"] },\n  { id:\"faq-044\", kw:[\"where implemented\",\"countries\",\"which countries\",\"global presence\",\"کشورها\",\"países\",\"pays\",\"länder\",\"wo implementiert\"] },\n  { id:\"faq-045\", kw:[\"implemented in other countries\",\"partnerships\",\"ministries\",\"expand\",\"گسترش\",\"شراکت\",\"otros países\",\"outros países\",\"autres pays\",\"andere länder\",\"partnerschaften\"] },\n  { id:\"faq-046\", kw:[\"one-time pay\",\"once\",\"without abonment\",\"no subscription\",\"دفعة واحدة\", \"مرة واحدة\", \"بدون اشتراك\", \"بدون اشتراك\",\"pago único\", \"una sola vez\", \"sin abono\", \"sin suscripción\",\"paiement unique\", \"une seule fois\", \"sans abonnement\" , \"Einmalzahlung\", \"einmalig\", \"ohne Abo\", \"kein Abonnement\",\"eemol Bezuelung\", \"eemol\", \"ouni Ofbezuelung\", \"keen Abonnement\",\"pagamento único\", \"uma vez\", \"sem subscrição\", \"sem mensalidade\"] },\n];\n\n// 3) Guardrail: find direct keyword hits\nconst hits = [];\nfor (const m of MAP) {\n  if (m.kw.some(k => q.includes(k))) hits.push(m.id);\n}\n\n// 4) Shortlist:\n// - if guardrail hit(s): shortlist those IDs\n// - else: use lightweight token overlap to pick top-N (default 12)\nlet shortlist = [];\nif (hits.length) {\n  const set = new Set(hits);\n  shortlist = faqs.filter(f => set.has(f.id)).map(f => ({ id: f.id, q: f.q }));\n} else {\n  const toks = q.split(/\\s+/).filter(t => t && t.length > 2);\n  function score(text) {\n    const t = text.toLowerCase();\n    let s = 0;\n    for (const tok of toks) if (t.includes(tok)) s += 1;\n    return s;\n  }\n  const scored = faqs.map(f => ({ id: f.id, q: f.q, _s: score(f.q) }));\n  scored.sort((a,b)=>b._s - a._s);\n  const N = Math.min(12, scored.length || 12);\n  shortlist = scored.slice(0, N).map(({id,q}) => ({ id, q }));\n}\n\n// Safety: never send empty list to the LLM (send a tiny subset)\nif (!shortlist.length) {\n  shortlist = faqs.slice(0, 6).map(f => ({ id: f.id, q: f.q }));\n}\n\nreturn [{\n  user_question: qRaw,                    // English question\n  faq_index: shortlist,                   // Shortlist for the classifier\n  detected_source_language: userLang || \"en\",\n  faqs: data.faq,\n  faqs_count: faqs.length\n}];\n"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        -800,
        -320
      ],
      "id": "e3a05a27-1693-4355-9cd4-71b9d6aee94f",
      "name": "Code1"
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=User question:\n={{ $json.user_question }}\n\nFAQ list (JSON):\n={{ JSON.stringify($json.faq_index) }}\n",
        "options": {
          "systemMessage": "You are a strict FAQ classifier for Magrid.\n\nINPUTS:\n- user_question: a short, possibly noisy user query\n- faq_index: a JSON array of {id, q}, already shortlisted\n\nYOUR TASK:\n1) Select the SINGLE best matching FAQ id from faq_index.\n2) If none is clearly relevant, return id \"none\".\n3) For extremely short queries (≤2 tokens), require a direct keyword/synonym overlap with one of the FAQ questions; otherwise return \"none\".\n4) Be conservative. Do not guess based on generic words.\n5) Return ONLY valid JSON:\n   {\"id\":\"<faq-id-or-none>\", \"reason\":\"<brief>\", \"confidence\": <0..1>}\n\nEXAMPLES:\nUser: \"language?\" \n→ {\"id\":\"faq-036\",\"reason\":\"Asks about supported languages; faq-036 covers language availability.\",\"confidence\":0.95}\n\nUser: \"what languages do you have in the app?\"\n→ {\"id\":\"faq-036\",\"reason\":\"Directly about supported languages.\",\"confidence\":0.95}\n\nUser: \"multilingual classroom?\"\n→ {\"id\":\"faq-008\",\"reason\":\"Asks about multilingual environments; faq-008 addresses this.\",\"confidence\":0.9}\n\nUser: \"pricing / cost\"\n→ {\"id\":\"faq-033\",\"reason\":\"Asks about cost; faq-033 covers pricing.\",\"confidence\":0.9}\n\nUser: \"devices?\"\n→ {\"id\":\"faq-034\",\"reason\":\"Asks about supported devices.\",\"confidence\":0.95}\n\nUser: \"offline / internet needed?\"\n→ {\"id\":\"faq-035\",\"reason\":\"Connectivity/offline usage; faq-035.\",\"confidence\":0.95}\n\nUser: \"ages / grades?\"\n→ {\"id\":\"faq-006\",\"reason\":\"Supported ages/grades.\",\"confidence\":0.95}\n\nUser: \"return policy\" \n→ {\"id\":\"none\",\"reason\":\"Not in Magrid FAQ list (store-like topic).\",\"confidence\":0.0}\n\nUser: \"?\"\n→ {\"id\":\"none\",\"reason\":\"No meaningful query.\",\"confidence\":0.0}\n"
        }
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 2.2,
      "position": [
        -464,
        -320
      ],
      "id": "b02f25f6-fd5c-4173-a4bc-9cc4eb0d2370",
      "name": "AI Agent"
    },
    {
      "parameters": {
        "model": {
          "__rl": true,
          "value": "gpt-5",
          "mode": "list",
          "cachedResultName": "gpt-5"
        },
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.2,
      "position": [
        -464,
        -112
      ],
      "id": "255781b5-c0cc-47fb-85c6-7a24ba1122ec",
      "name": "OpenAI Chat Model",
      "credentials": {
        "openAiApi": {
          "id": "WYx43EFCfNuobSYP",
          "name": "OpenAi account 2"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "// Code2 (FAQ_Init)\n// Save all FAQs into workflow static data\n\nconst data = $getWorkflowStaticData('global');\n\nif (!Array.isArray(data.faq) || data.faq.length === 0) {\n  data.faq = [\n    { id: \"faq-001\", q: \"What is Magrid and who is it for?\", a: \"Magrid is an inclusive, language-free digital platform that teaches early mathematics and cognitive skills to children from 4 years old. It is ideal for schools, families, special education programs and NGOs with social impact projects.\" },\n    { id: \"faq-002\", q: \"How is Magrid different from other programs?\", a: \"Unlike traditional platforms, Magrid uses visual instructions and requires no reading or spoken language. It is grounded in neuroscience and designed to support children with diverse learning needs.\" },\n    { id: \"faq-003\", q: \"What skills does Magrid develop?\", a: \"Magrid helps children improve problem-solving, memory, spatial awareness, and logical thinking—without needing to read.\" },\n    { id: \"faq-004\", q: \"Is Magrid based on scientific research?\", a: \"Yes. Magrid is backed by over 10 years of research in neuroscience, child development, and psychology. It is supported by six published studies, with more ongoing, led by the University of Luxembourg in collaboration with universities in Germany, France, and Portugal.\" },\n    { id: \"faq-005\", q: \"Does Magrid include assessments to track student progress?\", a: \"Yes. Magrid features a continuous assessment system with three key assessments: a pre-test, a mid-test, and a post-test. These track progress and support data-driven instruction.\" },\n    { id: \"faq-006\", q: \"What age or grade levels does Magrid support?\", a: \"Magrid is designed for Early Childhood Education and early Primary School (approximately ages 4 to 8) and beyond this age for children with special educational needs and different developmental needs.\" },\n\n    { id: \"faq-007\", q: \"Why is Magrid considered inclusive?\", a: \"Magrid’s language-free design ensures accessibility for all learners, including those with learning difficulties, special needs, or limited literacy.\" },\n    { id: \"faq-008\", q: \"Can Magrid be used in multilingual environments?\", a: \"Yes, Magrid is ideal for multilingual classrooms as it relies on visual instructions, making it accessible regardless of the child's spoken language.\" },\n    { id: \"faq-009\", q: \"Does Magrid support children with special educational needs?\", a: \"Yes, Magrid is designed to adapt to a wide range of learners, including children with developmental delays, cognitive challenges, and other learning disabilities.\" },\n    { id: \"faq-010\", q: \"Is Magrid suitable for children with autism or ADHD?\", a: \"Yes. Magrid is sensory-friendly and intentionally designed without overstimulating elements or gamified reward systems. It fosters positive learning through gentle reinforcement, allows children to progress at their own pace, and avoids penalizing mistakes—reducing frustration and supporting self-confidence.\" },\n    { id: \"faq-011\", q: \"Can children with hearing impairments use Magrid?\", a: \"Absolutely. Magrid is mostly visual and doesn’t rely on spoken instructions, making it effective for children with hearing loss. A small number of activities include optional audio, but these can be skipped without affecting progress.\" },\n\n    { id: \"faq-012\", q: \"How can Magrid be implemented in schools or learning centers?\", a: \"Magrid can be used on tablets or touch screen devices. We provide training, onboarding sessions, and ongoing support to help educators integrate Magrid into their teaching.\" },\n    { id: \"faq-013\", q: \"Does Magrid align with existing curricula?\", a: \"Yes. Magrid builds key foundational skills that fit into any curriculum. It is already mapped to Common Core, IB, and several national standards, with curriculum alignment guides available.\" },\n    { id: \"faq-014\", q: \"Can I implement Magrid at any time during the school year?\", a: \"Absolutely. Magrid can be integrated at any point in the academic year, making it a flexible support tool to reinforce math concepts and cognitive development.\" },\n    { id: \"faq-015\", q: \"How long does the Magrid program take to complete?\", a: \"There is no fixed duration. Magrid is self-paced and adapts to each child’s learning speed, allowing them to progress based on their individual development.\" },\n    { id: \"faq-016\", q: \"How often should students use Magrid?\", a: \"We recommend 1 hour of use per week, delivered in 15–20 minute sessions, 3 to 4 times per week. The screen time can be adjusted in the app’s settings.\" },\n    { id: \"faq-017\", q: \"Is teacher training required?\", a: \"Training is not required but strongly recommended. We provide free onboarding, quick-start guides, and live sessions for teachers.\" },\n    { id: \"faq-018\", q: \"How is student progress monitored?\", a: \"Teachers and administrators have access to real-time progress reports showing learning paths, strengths, and areas for improvement. Basic analytics are available within the app, while detailed class- and school-level reports are provided through the teacher/admin website panel.\" },\n    { id: \"faq-019\", q: \"How does Magrid support early detection of learning difficulties?\", a: \"Through its assessment system and detailed progress reports, Magrid helps identify early signs of challenges such as dyslexia, dysgraphia, and dyscalculia, making it a valuable pre-screening tool.\" },\n    { id: \"faq-020\", q: \"Can Magrid be used in early intervention or remedial programs?\", a: \"Absolutely. Its step-by-step progression and adaptive design make Magrid especially useful for early intervention and remedial instruction.\" },\n    { id: \"faq-021\", q: \"Which institutions can use Magrid?\", a: \"Magrid is used in schools, preschools, NGOs, inclusive education programs, and community centers.\" },\n    { id: \"faq-022\", q: \"How does Magrid promote responsible technology use?\", a: \"Magrid is designed to align with the World Health Organization’s recommendations on screen time for young children. We limit use to short, focused sessions — no more than 1 hour per week — to ensure healthy digital habits and balanced development.\" },\n\n    { id: \"faq-023\", q: \"Can I use Magrid at home?\", a: \"Yes. Magrid is suitable for home use and homeschooling. It provides an independent learning journey supported by parental insights.\" },\n    { id: \"faq-024\", q: \"How do I get access as a parent?\", a: \"Parents can download the Magrid App from the different App stores: Apple Store, Google Play and Microsoft Store. We offer licenses for home use. Contact us via our website if you need access.\" },\n    { id: \"faq-025\", q: \"Can multiple children use the same device?\", a: \"Yes, you can create multiple child profiles on a single device.\" },\n    { id: \"faq-026\", q: \"Is Magrid safe for children to use alone?\", a: \"Absolutely. The app has no ads, no external links, and complies with child safety standards (GDPR, COPPA).\" },\n    { id: \"faq-027\", q: \"How can I track my child’s progress?\", a: \"Parents receive regular progress updates and can view detailed reports via the app.\" },\n\n    { id: \"faq-028\", q: \"What types of licenses are available?\", a: \"We offer licenses for schools, NGOs, institutions, and families. Licenses can be customized based on the number of students and duration. Check our different subscriptions here.\" },\n    { id: \"faq-029\", q: \"Can I try Magrid before purchasing?\", a: \"Yes. We offer free demos and trial access.\" },\n    { id: \"faq-030\", q: \"Can I upgrade or change my license?\", a: \"Yes, licenses are flexible and scalable at any point during the subscription.\" },\n    { id: \"faq-031\", q: \"How are user accounts managed?\", a: \"Student profiles can be created and managed by teachers, administrators, or parents.\" },\n    { id: \"faq-032\", q: \"Does Magrid comply with data protection regulations?\", a: \"Yes. Magrid strictly adheres to GDPR and other international data protection laws, as outlined in our Terms & Conditions.\" },\n    { id: \"faq-033\", q: \"How much does Magrid cost?\", a: \"Parents: Pricing is region-specific and adapted to local exchange rates within each app store. Flexible plans are available (3 or 12 months), and each subscription covers all children within a family account. Educators: single teacher account for small groups, or tailored solutions for larger classes. Schools/organizations: institutional packages (from ~15 students) with full teacher training, 24/7 support, offline resources, and impact reports. you can also register and check in your account(https://panel.magrid.education/Account/Welcome) or chat with our support in whatsapp and ask: +352 661 337704\" },\n\n    { id: \"faq-034\", q: \"On which devices can Magrid be used?\", a: \"Magrid works on Android and iOS (Apple) tablets, and computers. Tablets and digital devices with touch screens are recommended.\" },\n    { id: \"faq-035\", q: \"Does Magrid require internet access?\", a: \"An internet connection is only needed to download the app and receive updates. Students can use Magrid offline for all learning activities. Schools need to connect the devices to the internet regularly (at least once a week) to sync their students’ progress.\" },\n    { id: \"faq-036\", q: \"Is Magrid available in multiple languages?\", a: \"Yes. Magrid is currently available in English, Spanish, French, German, Portuguese, Luxembourgish and Nepali.\" },\n    { id: \"faq-037\", q: \"Is technical support available?\", a: \"Yes. We offer Whatsapp and email support 365 days.\" },\n\n    { id: \"faq-038\", q: \"How quickly can Magrid be implemented in a classroom or center?\", a: \"Magrid can be set up and used within a single day. With quick-start guides and optional free onboarding sessions, educators can start working with their students right away.\" },\n    { id: \"faq-039\", q: \"What kind of training and support is available for educators?\", a: \"We offer live onboarding sessions, training materials, and continued support via email and Whatsapp. Our team is available throughout the implementation to ensure a smooth experience.\" },\n    { id: \"faq-040\", q: \"How does Magrid adapt to different classroom contexts (rural, urban, low-resource)?\", a: \"Magrid is flexible and scalable. Its offline capabilities and visual design make it effective in both low-resource settings and well-equipped classrooms.\" },\n    { id: \"faq-041\", q: \"Can Magrid be used to complement other math programs?\", a: \"Yes. Magrid is compatible with most early math curricula and is often used as a complementary tool to strengthen foundational math and cognitive skills.\" },\n    { id: \"faq-042\", q: \"How does Magrid support monitoring at scale (e.g., district or NGO level)?\", a: \"Administrators can access dashboards with aggregated data across schools or regions, allowing for easy tracking of impact, usage, and student progress.\" },\n    { id: \"faq-043\", q: \"Does Magrid offer any tools for reporting outcomes to stakeholders?\", a: \"Yes. We provide progress reports that can be exported and shared with parents, school leadership, or funding partners.\" },\n\n    { id: \"faq-044\", q: \"Where is Magrid currently implemented?\", a: \"Magrid is being used in schools and communities across 10+ countries, including Luxembourg, Portugal, Nepal, Peru, Brazil, Colombia, Curaçao, Ecuador, India, Ukraine, Laos... and others joining in 2025.\" },\n    { id: \"faq-045\", q: \"Can Magrid be implemented in other countries?\", a: \"Yes. We partner with local institutions, schools, NGOs, and ministries of education to bring Magrid to new communities. Contact us to explore opportunities: contact@magrid.education.\" },\n    { id: \"faq-048\", q: \"What are the rates for families?\" , a:\"Parents: Pricing is region-specific and adapted to local exchange rates within each app store. Flexible plans are available (3 or 12 months), and each subscription covers all children within a family account. Educators: single teacher account for small groups, or tailored solutions for larger classes. Schools/organizations: institutional packages (from ~15 students) with full teacher training, 24/7 support, offline resources, and impact reports. you can also register and check in your account(https://panel.magrid.education/Account/Welcome) or chat with our support in whatsapp and ask: +352 661 337704\"},\n    { id: \"faq-046\", q: \"possibilities to buy the access once, without an abo?\", a: \"Yes, a one-time payment option is available. Please contact contact@magrid.education, and they will assist you with the process.\" }\n  ];\n}\n\n// مهم: ورودی قبلی (chatInput) رو هم پاس بدیم\nreturn [{ ...$json, faqs: data.faq, ok: true, count: data.faq.length }];\n"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        -1072,
        -320
      ],
      "id": "11f00409-13e5-4cd2-9e22-67b24c27c22c",
      "name": "Code4"
    },
    {
      "parameters": {
        "text": "={{ $json.faq_answer_en }}",
        "translateTo": "={{ $('Translate a language2').item.json.detected_source_language }}",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.deepL",
      "typeVersion": 1,
      "position": [
        -1424,
        -960
      ],
      "id": "5b330d81-9444-448c-822a-c6533c43c3b0",
      "name": "Translate a language3",
      "credentials": {
        "deepLApi": {
          "id": "a9rJcw5hVBATnLJS",
          "name": "DeepL account 2"
        }
      }
    },
    {
      "parameters": {
        "authentication": "webhook",
        "content": "={{ $json.chatInput }} {{ $json.output }} {{ $json.text }}",
        "options": {}
      },
      "type": "n8n-nodes-base.discord",
      "typeVersion": 2,
      "position": [
        -2608,
        -1024
      ],
      "id": "36ed01d7-0e33-4cf2-8dd2-437671a12deb",
      "name": "Discord2",
      "webhookId": "0eb5e71a-5272-4a29-8912-2ab78da05d7b",
      "credentials": {
        "discordWebhookApi": {
          "id": "LpkTAbVqO0s6RZFK",
          "name": "Discord Webhook account 2"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "let parsed = {};\ntry {\n  parsed = JSON.parse($json.output || \"{}\");\n} catch(e) {\n  parsed = {};\n}\n\nconst id = parsed.id || \"none\";\n\nconst faqMap = {\n  \"faq-001\": \"Magrid is an inclusive, language-free digital platform that teaches early mathematics and cognitive skills to children from 4 years old. It is ideal for schools, families, special education programs and NGOs with social impact projects.\",\n  \"faq-002\": \"Unlike traditional platforms, Magrid uses visual instructions and requires no reading or spoken language. It is grounded in neuroscience and designed to support children with diverse learning needs.\",\n  \"faq-003\": \"Magrid helps children improve problem-solving, memory, spatial awareness, and logical thinking—without needing to read.\",\n  \"faq-004\": \"Magrid is backed by over 10 years of research in neuroscience, child development, and psychology. It is supported by six published studies, with more ongoing, led by the University of Luxembourg in collaboration with universities in Germany, France, and Portugal.\",\n  \"faq-005\": \"Magrid features a continuous assessment system with three key assessments: a pre-test, a mid-test, and a post-test. These track progress and support data-driven instruction.\",\n  \"faq-006\": \"Magrid is designed for Early Childhood Education and early Primary School (approximately ages 4 to 8) and beyond this age for children with special educational needs and different developmental needs.\",\n  \"faq-007\": \"Magrid’s language-free design ensures accessibility for all learners, including those with learning difficulties, special needs, or limited literacy.\",\n  \"faq-008\": \"Magrid is ideal for multilingual classrooms as it relies on visual instructions, making it accessible regardless of the child's spoken language.\",\n  \"faq-009\": \"Magrid is designed to adapt to a wide range of learners, including children with developmental delays, cognitive challenges, and other learning disabilities.\",\n  \"faq-010\": \"Magrid is sensory-friendly and intentionally designed without overstimulating elements or gamified reward systems. It fosters positive learning through gentle reinforcement, allows children to progress at their own pace, and avoids penalizing mistakes—reducing frustration and supporting self-confidence.\",\n  \"faq-011\": \"Absolutely. Magrid is mostly visual and doesn’t rely on spoken instructions, making it effective for children with hearing loss. A small number of activities include optional audio, but these can be skipped without affecting progress.\",\n  \"faq-012\": \"Magrid can be used on tablets or touch screen devices. We provide training, onboarding sessions, and ongoing support to help educators integrate Magrid into their teaching.\",\n  \"faq-013\": \"Magrid builds key foundational skills that fit into any curriculum. It is already mapped to Common Core, IB, and several national standards, with curriculum alignment guides available.\",\n  \"faq-014\": \"Absolutely. Magrid can be integrated at any point in the academic year, making it a flexible support tool to reinforce math concepts and cognitive development.\",\n  \"faq-015\": \"There is no fixed duration. Magrid is self-paced and adapts to each child’s learning speed, allowing them to progress based on their individual development.\",\n  \"faq-016\": \"We recommend 1 hour of use per week, delivered in 15–20 minute sessions, 3 to 4 times per week. The screen time can be adjusted in the app’s settings.\",\n  \"faq-017\": \"Training is not required but strongly recommended. We provide free onboarding, quick-start guides, and live sessions for teachers.\",\n  \"faq-018\": \"Teachers and administrators have access to real-time progress reports showing learning paths, strengths, and areas for improvement. Basic analytics are available within the app, while detailed class- and school-level reports are provided through the teacher/admin website panel.\",\n  \"faq-019\": \"Through its assessment system and detailed progress reports, Magrid helps identify early signs of challenges such as dyslexia, dysgraphia, and dyscalculia, making it a valuable pre-screening tool.\",\n  \"faq-020\": \"Absolutely. Its step-by-step progression and adaptive design make Magrid especially useful for early intervention and remedial instruction.\",\n  \"faq-021\": \"Magrid is used in schools, preschools, NGOs, inclusive education programs, and community centers.\",\n  \"faq-022\": \"Magrid is designed to align with the World Health Organization’s recommendations on screen time for young children. We limit use to short, focused sessions — no more than 1 hour per week — to ensure healthy digital habits and balanced development.\",\n  \"faq-023\": \"Magrid is suitable for home use and homeschooling. It provides an independent learning journey supported by parental insights.\",\n  \"faq-024\": \"Parents can download the Magrid App from the different App stores: Apple Store, Google Play and Microsoft Store. We offer licenses for home use. Contact us via our website if you need access.\",\n  \"faq-025\": \"you can create multiple child profiles on a single device.\",\n  \"faq-026\": \"Absolutely. The app has no ads, no external links, and complies with child safety standards (GDPR, COPPA).\",\n  \"faq-027\": \"Parents receive regular progress updates and can view detailed reports via the app.\",\n  \"faq-028\": \"We offer licenses for schools, NGOs, institutions, and families. Licenses can be customized based on the number of students and duration. Check our different subscriptions here.\",\n  \"faq-029\": \"We offer free demos and trial access.\",\n  \"faq-030\": \"licenses are flexible and scalable at any point during the subscription.\",\n  \"faq-031\": \"Student profiles can be created and managed by teachers, administrators, or parents.\",\n  \"faq-032\": \"Magrid strictly adheres to GDPR and other international data protection laws, as outlined in our Terms & Conditions.\",\n  \"faq-033\": \"Parents: Pricing is region-specific and adapted to local exchange rates within each app store. Flexible plans are available (3 or 12 months), and each subscription covers all children within a family account. Educators: single teacher account for small groups, or tailored solutions for larger classes. Schools/organizations: institutional packages (from ~15 students) with full teacher training, 24/7 support, offline resources, and impact reports. you can also register and check in your account(https://panel.magrid.education/Account/Welcome) or chat with our support in whatsapp and ask: +352 661 337704\",\n  \"faq-034\": \"Magrid works on Android and iOS (Apple) tablets, and computers. Tablets and digital devices with touch screens are recommended.\",\n  \"faq-035\": \"An internet connection is only needed to download the app and receive updates. Students can use Magrid offline for all learning activities. Schools need to connect the devices to the internet regularly (at least once a week) to sync their students’ progress.\",\n  \"faq-036\": \"Magrid is currently available in English, Spanish, French, German, Portuguese, Luxembourgish and Nepali.\",\n  \"faq-037\": \"We offer WhatsApp and email support 365 days a year. you can also send WhatsApp message to +352661337704\",\n  \"faq-038\": \"Magrid can be set up and used within a single day. With quick-start guides and optional free onboarding sessions, educators can start working with their students right away.\",\n  \"faq-039\": \"We offer live onboarding sessions, training materials, and continued support via email and WhatsApp. Our team is available throughout the implementation to ensure a smooth experience.\",\n  \"faq-040\": \"Magrid is flexible and scalable. Its offline capabilities and visual design make it effective in both low-resource settings and well-equipped classrooms.\",\n  \"faq-041\": \"Magrid is compatible with most early math curricula and is often used as a complementary tool to strengthen foundational math and cognitive skills.\",\n  \"faq-042\": \"Administrators can access dashboards with aggregated data across schools or regions, allowing for easy tracking of impact, usage, and student progress.\",\n  \"faq-043\": \"We provide progress reports that can be exported and shared with parents, school leadership, or funding partners.\",\n  \"faq-044\": \"Magrid is being used in schools and communities across 10+ countries, including: Luxembourg, Portugal, Nepal, Peru, Brazil, Colombia, Curaçao, Ecuador, India, Ukraine, Laos... and others joining in 2025.\",\n  \"faq-046\": \"Hello\",\n  \"faq-045\": \"We partner with local institutions, schools, NGOs, and ministries of education to bring Magrid to new communities. Contact us to explore opportunities.\",\n  \"faq-046\": \"Yes, a one-time payment option is available. Please contact contact@magrid.education, and they will assist you with the process.\",\n  \"faq-047\": \"Hello, How can I help you? 😊\"\n};\n\nlet answer = \"🫤 Oops—No answer available at the moment. Share your Email 📧 or contact us on 💬 WhatsApp +352661337704 and we’ll reply soon ✌🏻!\";\nif (faqMap[id]) {\n  answer = faqMap[id];\n}\n\nreturn [{\n  ...$json,\n  faq_answer_en: answer,\n}];\n"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        -64,
        -320
      ],
      "id": "b0f90942-ac8a-484e-a29b-da1d434fb14b",
      "name": "Code5"
    },
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "c0001645-7703-4590-8179-4de420c492d3",
        "responseMode": "responseNode",
        "options": {}
      },
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2.1,
      "position": [
        -1584,
        -544
      ],
      "id": "67a16335-8218-4e7e-82c1-e4c181b254e4",
      "name": "Webhook1",
      "webhookId": "c0001645-7703-4590-8179-4de420c492d3"
    },
    {
      "parameters": {},
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.3,
      "position": [
        -368,
        -112
      ],
      "id": "33599718-536a-434f-bd13-b4bea6966490",
      "name": "Simple Memory1"
    },
    {
      "parameters": {
        "mode": "chooseBranch"
      },
      "type": "n8n-nodes-base.merge",
      "typeVersion": 3.2,
      "position": [
        176,
        -304
      ],
      "id": "4b0f34bf-9b85-4ff9-8787-240c4e326eb9",
      "name": "Merge2"
    },
    {
      "parameters": {
        "authentication": "webhook",
        "content": "={{ $('When chat message received').item.json.chatInput }}{{ \n\" ==> Question  \" + ($json.chatInput ?? \"\") + \n\"\\n\\nAnswer: \" + ($json.faq_answer_en ?? \"\") \n}}\n",
        "options": {}
      },
      "type": "n8n-nodes-base.discord",
      "typeVersion": 2,
      "position": [
        208,
        32
      ],
      "id": "baf58138-ecb0-409e-b71e-4360f8f0da73",
      "name": "Discord3",
      "webhookId": "0eb5e71a-5272-4a29-8912-2ab78da05d7b",
      "credentials": {
        "discordWebhookApi": {
          "id": "q9CB6zSWBEuVHGJt",
          "name": "Discord Webhook account 5"
        }
      }
    },
    {
      "parameters": {
        "mode": "chooseBranch",
        "useDataOfInput": "={{ 1 }}"
      },
      "type": "n8n-nodes-base.merge",
      "typeVersion": 3.2,
      "position": [
        -432,
        -976
      ],
      "id": "dc10ec96-39c8-45a9-86e6-541b235532a3",
      "name": "Merge3"
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api-free.deepl.com/v2/translate",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "DeepL-Auth-Key 4aa052d6-3dd6-47ed-a705-2008eed0a004:fx"
            }
          ]
        },
        "sendBody": true,
        "contentType": "form-urlencoded",
        "bodyParameters": {
          "parameters": [
            {
              "name": "text",
              "value": "={{$json.chatInput}}"
            },
            {
              "name": "target_lang",
              "value": "EN"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.4,
      "position": [
        -1136,
        -960
      ],
      "id": "b4c3fdea-a091-4c2f-ae54-aefc107b4e26",
      "name": "HTTP Request2"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "0c6c60b6-eddb-4db9-802f-b415db204a1f",
              "name": "respond",
              "value": "={{ $('Code5').item.json.faq_answer_en }}",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        720,
        -48
      ],
      "id": "a4acf780-f043-4bdd-a2ea-db1627488285",
      "name": "Edit Fields"
    },
    {
      "parameters": {
        "respondWith": "text",
        "responseBody": "={{ $json.faq_answer_en }}",
        "options": {}
      },
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1.5,
      "position": [
        480,
        -304
      ],
      "id": "5e0a6576-9983-448d-a8a3-69bd887278f1",
      "name": "Respond to Webhook"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.noOp",
      "typeVersion": 1,
      "position": [
        432,
        32
      ],
      "id": "4059e14a-478c-4391-9ad5-45558ef6ddcc",
      "name": "No Operation, do nothing"
    }
  ],
  "pinData": {},
  "connections": {
    "When chat message received": {
      "main": [
        [
          {
            "node": "Code4",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Translate a language": {
      "main": [
        []
      ]
    },
    "Translate a language1": {
      "main": [
        []
      ]
    },
    "Code1": {
      "main": [
        [
          {
            "node": "AI Agent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "AI Agent": {
      "main": [
        [
          {
            "node": "Code5",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "OpenAI Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "AI Agent",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Code4": {
      "main": [
        [
          {
            "node": "Code1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Translate a language3": {
      "main": [
        []
      ]
    },
    "Discord2": {
      "main": [
        []
      ]
    },
    "Code5": {
      "main": [
        [
          {
            "node": "Merge2",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Webhook1": {
      "main": [
        [
          {
            "node": "Code4",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Simple Memory1": {
      "ai_memory": [
        [
          {
            "node": "AI Agent",
            "type": "ai_memory",
            "index": 0
          }
        ]
      ]
    },
    "Merge2": {
      "main": [
        [
          {
            "node": "Respond to Webhook",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Discord3": {
      "main": [
        [
          {
            "node": "No Operation, do nothing",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "HTTP Request2": {
      "main": [
        []
      ]
    },
    "Merge3": {
      "main": [
        []
      ]
    },
    "Edit Fields": {
      "main": [
        []
      ]
    },
    "Respond to Webhook": {
      "main": [
        [
          {
            "node": "Discord3",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": true,
  "settings": {
    "executionOrder": "v1"
  },
  "versionId": "9229e0f2-1992-4361-82e2-7acc1c814136",
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "e277aaeb25adaa67f5f8cb9dae56e3aeabf19f993fd064fc3ed2591aca3f5042"
  },
  "id": "vaNhsX2BjFnEQbLj",
  "tags": []
}
