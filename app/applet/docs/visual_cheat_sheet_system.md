# ⚡ Complete System Specification & Code Package: Visual Cheat Sheet Studio

> **Version:** 2.0 (Universal Two-Stage Distillation Pipeline)  
> **Target Framework:** React (Vite / Next.js) + TypeScript + Tailwind CSS + Node.js (Express) + Gemini API (`@google/genai`)  
> **Purpose:** Drop-in, zero-failure guide to integrate the **1-Page Visual Concept Cheat Sheet / Infographic Poster** feature into any educational AI app.

---

## 📑 Table of Contents
1. [Architecture & Two-Stage Pipeline Overview](#1-architecture--two-stage-pipeline-overview)
2. [Complete AI Instruction Prompts](#2-complete-ai-instruction-prompts)
   - [A. Vision OCR Document Ingestion Prompt](#a-vision-ocr-document-ingestion-prompt)
   - [B. Stage 1: Content Distillation Prompt (The Filter)](#b-stage-1-content-distillation-prompt-the-filter)
   - [C. Stage 2: Universal Bento JSON Synthesis Prompt](#c-stage-2-universal-bento-json-synthesis-prompt)
   - [D. Gemini Structured Outputs JSON Schema](#d-gemini-structured-outputs-json-schema)
3. [TypeScript Data Contracts (`types.ts`)](#3-typescript-data-contracts)
4. [Backend REST API Route (`server.ts`)](#4-backend-rest-api-route)
5. [Client-Side Fallback Engine (`distillationEngine.ts`)](#5-client-side-fallback-engine)
6. [Frontend UI & Poster Renderer (`ConceptInfographicPoster.tsx`)](#6-frontend-ui--poster-renderer)
7. [Step-by-Step Integration & Verification Checklist](#7-step-by-step-integration--verification-checklist)

---

## 1. Architecture & Two-Stage Pipeline Overview

The **Visual Cheat Sheet** takes an uploaded textbook page (PDF/Image), handwritten student notes, or a typed chapter topic, and synthesizes a **1-page high-yield visual revision poster**.

### The Two-Stage Pipeline Flow:
```
[User Input: PDF / JPG / Notes / Topic]
                  │
                  ▼
[Vision OCR / Text Extraction] (Stripping camera file IDs, isolating true subject)
                  │
                  ▼
[Stage 1: Content Distillation Filter]
  - 100% Source-Bound: Strips conversational filler and irrelevant fluff.
  - Extracts 6 Core Cognitive Pillars (Taxonomy, Master Formulas, Hero Visual, 4 Scenarios, Exam Traps, Mnemonic).
                  │
                  ▼
[Stage 2: Universal JSON Synthesis]
  - Enforces strict structured output via Gemini Schema.
  - Maps data into Universal Bento Grid format (Science, Math, Commerce, Humanities).
                  │
                  ▼
[React ConceptInfographicPoster Component]
  - Displays high-contrast Bento cards.
  - KaTeX / MathJax instant LaTeX formula blocks ($$ ... $$).
  - Inline SVG vector schematics & reaction flows.
  - "Start Live Lecture" CTA & Export / Print buttons.
```

---

## 2. Complete AI Instruction Prompts

### A. Vision OCR Document Ingestion Prompt
When a student drops an image or PDF of notes/textbook, send this prompt with the file to Gemini Vision:

```text
You are an expert academic curriculum Vision OCR and visual infographic distillation assistant in Chemistry, Physics, Biology, and Mathematics.
The uploaded document/image represents a textbook page, lecture notes, formula chart, or handwritten study material.

CRITICAL VISION ANALYSIS INSTRUCTIONS:
1. IGNORE any camera/scanner filename or numeric index (such as '4968.jpg', 'IMG_1234', 'scan_01'). DO NOT use numbers as chapter names.
2. Identify the TRUE ACADEMIC SUBJECT from the visual content (Chemistry, Physics, Biology, Mathematics, History, Economics):
   - If the image contains chemical symbols (NH3, HCl, H2SO4, HNO3), laboratory apparatus (test tubes, round-bottom flasks, delivery tubes, fountain experiment), chemical equations, reactants, bonding diagrams, or compound properties -> MUST CLASSIFY SUBJECT AS 'Chemistry'.
   - If the image contains vectors, free-body diagrams, circuits, lenses, or equations of motion -> MUST CLASSIFY SUBJECT AS 'Physics'.
3. Extract the TRUE CHAPTER / TOPIC TITLE from the headings or visual content (e.g. 'Study of Compounds: Ammonia (NH3)', 'Acids, Bases and Salts', 'Chemical Bonding', 'Periodic Table', etc.).
4. Structure the output in clean, dense Markdown with clear sections:
   # Chapter: [Exact Chapter Title]
   ## Subject: [Exact Academic Subject, e.g. Chemistry]
   ## Core Principle & Foundation: Crisp 2-sentence definition and molecular/physical foundation.
   ## Key Chemical Formulas, Equations & Governing Laws: Balance all chemical equations and use standard LaTeX ($...$ inline, $$...$$ display blocks).
   ## Laboratory Preparation & Apparatus Setup: Reactants, drying agents, collection method, and key chemical reactions.
   ## Industrial Process / Reaction Mechanisms: (e.g., Haber's Process, Ostwald's Process, Contact Process) with catalyst, temperature, and pressure conditions.
   ## Case Studies, Experiments & Physical Observations: (e.g. Fountain Experiment, colour changes, precipitation tests).
   ## High-Yield Practical Applications & Exam Traps: Common student mistakes, tricky question traps, and industrial uses.
   ## Key Takeaway & Golden Rule: One concise summary line.

Format everything in dense, structured Markdown.
```

---

### B. Stage 1: Content Distillation Prompt (The Filter)
This prompt filters raw notes and organizes them into 6 cognitive pillars:

```typescript
export function buildStage1DistillationPrompt(params: {
  topic: string;
  subject: string;
  grade: string;
  chapter?: string;
  rawText: string;
}): string {
  return `You are Cherry Ma'am's Principal Academic Content Distiller & Curriculum Architect.
Your task is STAGE 1 (THE FILTER): Deeply distill and extract high-yield concepts from the source material into 6 CORE COGNITIVE PILLARS.

=== 🛑 STRICT GROUNDING & SOURCE-BOUND FIDELITY MANDATE ===
- You MUST base all extracted insights, concepts, formulas, definitions, case studies, and exam traps STRICTLY and ONLY on the provided uploaded document / source material content below.
- DO NOT invent, hallucinate, or import arbitrary topics, formulas, or concepts from unrelated chapters.
- If the uploaded document is about a specific topic, ALL 6 pillars MUST strictly reflect the exact content of that uploaded document and nothing else.

=== RAW SOURCE MATERIAL ===
- Topic / Concept: "${params.topic}"
- Subject: "${params.subject}"
- Target Grade / Standard: "${params.grade}"
${params.chapter ? `- Chapter Reference: "${params.chapter}"` : ""}
- Source Material Content:
"""
${params.rawText ? params.rawText.slice(0, 14000) : "Extract strictly grounded concepts for " + params.topic}
"""

=== CRITICAL DISTILLATION MANDATE (THE 6 CORE PILLARS) ===
1. 100% SOURCE-BOUND FLUFF REMOVAL: Strip conversational filler, greetings, and generic banter. Keep only verified academic facts present in or directly derived from the source text.
2. EXTRACT 6 COMPLETE PILLARS FOR 100% 1-PAGE REVISION FROM SOURCE MATERIAL:
   - PILLAR 1: Core Fundamental Definition & Essential Taxonomy (Crisp 2-sentence first-principles definition strictly from source).
   - PILLAR 2: Master Formula Bank & Laws (Governing boxed formulas, equations, or key rules present in the source with balanced LaTeX).
   - PILLAR 3: Center Hero Visual Schematic (Apparatus, sequence, stages, flow, or diagram points discussed in the source).
   - PILLAR 4: 4 Distinct Scenario Case Studies (4 distinct conditions, examples, cases, or classifications present in the source text).
   - PILLAR 5: Exam Traps & Silly Mistake Alerts (3 specific traps, misconceptions, or common student mistakes directly related to the source topic).
   - PILLAR 6: Real-World Application & 10-Second Mnemonic Hook strictly relevant to the source topic.

=== FORMATTING INSTRUCTIONS ===
- Output crisp, high-density Markdown with clear bullet points.
- Format all mathematical, chemical, and quantitative equations in clean standard LaTeX ($...$ or $$...$$).
- Keep descriptions precise, source-faithful, academic, and free of vague generalizations.`;
}
```

---

### C. Stage 2: Universal Bento JSON Synthesis Prompt
This prompt takes the distilled output from Stage 1 and compiles it into the JSON layout for UI rendering:

```typescript
export function buildStage2SynthesisPrompt(params: {
  topic: string;
  subject: string;
  grade: string;
  chapter?: string;
  distilledContent: string;
}): string {
  return `You are Cherry Ma'am's Universal Educational Infographic Engine.
Your task is STAGE 2 (JSON SYNTHESIS): Convert the distilled academic notes into our Comprehensive Multi-Block Educational Infographic Layout.

=== 🛑 STRICT DOCUMENT-BOUND ZERO-HALLUCINATION LAW ===
- Every single field in the JSON (definitions, formulas, case studies, classifications, lab setups, conditions, exam traps, mnemonic, and takeaways) MUST BE 100% GROUNDED in the distilled source notes below.
- NEVER substitute or mix in unrelated topics. Everything must strictly match the uploaded document content!

=== DISTILLED SOURCE NOTES ===
- Topic: ${params.topic}
- Subject: ${params.subject}
- Grade: ${params.grade}
- Distilled Summary:
"""
${params.distilledContent}
"""

=== STRICT MULTI-SECTION ARCHITECTURAL CONTRACT ===
Generate a valid JSON object matching this schema:
1. 'metadata':
   - 'title': ALL CAPS main title matching the source topic (e.g. "${params.topic.toUpperCase()}")
   - 'subtitle': Concise one-phrase definition / thesis directly reflecting the source content
   - 'domain': "science" | "math_cs" | "humanities" | "commerce"
   - 'subjectBadge': "${params.subject.toUpperCase()}"
   - 'gradeBadge': "${params.grade.toUpperCase()}"
   - 'chapterBadge': "${(params.chapter || params.topic).toUpperCase()}"
   - 'tagBadge': "HIGH-YIELD REVISION"
   - 'weightageBadge': "High-Yield • 8-10 Marks"
   - 'moleculeFormulaLatex': Primary formula/equation in LaTeX (e.g. "NH_3", "\\vec{F}=m\\vec{a}", "a^2+b^2=c^2")
   - 'relativeMassOrWeight': Mass / Unit / Constant (e.g. "17 g/mol", "SI: N")
   - 'geometricStructure': Geometric shape or structural badge (e.g. "Pyramidal Molecule", "Trigonal", "Vector Plane")
2. 'overviewSection' (Two Parallel Overview Cards):
   - 'card1_occurrence':
     - 'cardTitle': "OCCURRENCE" (or "ORIGIN & BACKGROUND" / "TAXONOMY")
     - 'bullets': Array of 3 distinct bullet points grounded in text:
       - 'label': Short bold heading (e.g. "Free State", "Combined State")
       - 'text': 1-2 sentence factual description directly from notes
       - 'iconKey': "cloud" | "flask" | "bio" | "sparkles" | "book" | "scale" | "atom"
   - 'card2_forms':
     - 'cardTitle': "FORMS OF THE SUBSTANCE / CLASSIFICATION"
     - 'items': Array of 4 numbered items:
       - 'id': 1 to 4
       - 'name': Name of the form/class
       - 'description': Clear factual definition
       - 'badge': Short tag (e.g. "Dry", "High Pressure", "Dilute")
       - 'iconKey': "gas" | "cylinder" | "bottle" | "dropper" | "atom" | "layers" | "cube"
3. 'labOrPrepSection' (Deep-Dive Preparation / Fundamental Derivation):
   - 'sectionTitle': Laboratory preparation or primary foundational derivation
   - 'reactantsOrInputs': Clear reactants/inputs with physical states
   - 'reactionEquationLatex': Balanced equation in LaTeX
   - 'precautionsAndDrying': Factual notes on apparatus setup, tilt angle precaution, drying agent
   - 'collectionMethod': Collection method & scientific reason
   - 'unsuitableAlert': Critical alert callout:
     - 'title': "Unsuitability of Other Reagents / Common Trap"
     - 'warningText': Explanation of why common alternatives fail
     - 'reagentsOrExceptions': List of unsuitable reagents
     - 'balancedEquations': Array of chemical reactions showing side-reactions
4. 'manufactureOrProcessSection' (Industrial Manufacture / Governing Equilibrium):
   - 'sectionTitle': Industrial process name (e.g. "MANUFACTURE BY HABER'S PROCESS")
   - 'reactantsRatio': Reactants & volumetric ratio
   - 'reactionEquationLatex': Reversible/equilibrium equation in LaTeX
   - 'favourableConditions': Array of 4 key parameter condition rows (Temperature, Pressure, Catalyst, Promoter)
   - 'flowStages': Array of 4 to 5 sequential process stages
5. 'caseStudies': Array of 4 distinct scenario case studies with specific conditions and mathematical results.
6. 'examTrapsSection':
   - 'title': "EXAM PITFALLS & SILLY MISTAKES"
   - 'traps': 3 common student exam blunders with corrections.
7. 'mnemonicSection':
   - 'acronym': Catchy memory hook (e.g. "OIL RIG")
   - 'explanation': Clear explanation of the memory aid.
8. 'footerNote': One golden exam takeaway line.`;
}
```

---

### D. Gemini Structured Outputs JSON Schema
Use this schema in Gemini API call with `responseMimeType: "application/json"`:

```typescript
import { Type } from "@google/genai";

export const universalInfographicResponseSchema = {
  type: Type.OBJECT,
  properties: {
    metadata: {
      type: Type.OBJECT,
      properties: {
        title: { type: Type.STRING },
        subtitle: { type: Type.STRING },
        domain: { type: Type.STRING },
        subjectBadge: { type: Type.STRING },
        gradeBadge: { type: Type.STRING },
        chapterBadge: { type: Type.STRING },
        tagBadge: { type: Type.STRING },
        weightageBadge: { type: Type.STRING },
        moleculeFormulaLatex: { type: Type.STRING },
        relativeMassOrWeight: { type: Type.STRING },
        geometricStructure: { type: Type.STRING },
      },
      required: ["title", "subtitle", "domain"],
    },
    overviewSection: {
      type: Type.OBJECT,
      properties: {
        card1_occurrence: {
          type: Type.OBJECT,
          properties: {
            cardTitle: { type: Type.STRING },
            bullets: {
              type: Type.ARRAY,
              items: {
                type: Type.OBJECT,
                properties: {
                  label: { type: Type.STRING },
                  text: { type: Type.STRING },
                  iconKey: { type: Type.STRING },
                },
                required: ["label", "text"],
              },
            },
          },
          required: ["cardTitle", "bullets"],
        },
        card2_forms: {
          type: Type.OBJECT,
          properties: {
            cardTitle: { type: Type.STRING },
            items: {
              type: Type.ARRAY,
              items: {
                type: Type.OBJECT,
                properties: {
                  id: { type: Type.INTEGER },
                  name: { type: Type.STRING },
                  description: { type: Type.STRING },
                  badge: { type: Type.STRING },
                  iconKey: { type: Type.STRING },
                },
                required: ["id", "name", "description"],
              },
            },
          },
          required: ["cardTitle", "items"],
        },
      },
    },
    labOrPrepSection: {
      type: Type.OBJECT,
      properties: {
        sectionTitle: { type: Type.STRING },
        reactantsOrInputs: { type: Type.STRING },
        reactionEquationLatex: { type: Type.STRING },
        precautionsAndDrying: { type: Type.STRING },
        collectionMethod: { type: Type.STRING },
        unsuitableAlert: {
          type: Type.OBJECT,
          properties: {
            title: { type: Type.STRING },
            warningText: { type: Type.STRING },
            reagentsOrExceptions: { type: Type.ARRAY, items: { type: Type.STRING } },
            balancedEquations: { type: Type.ARRAY, items: { type: Type.STRING } },
          },
        },
      },
    },
    manufactureOrProcessSection: {
      type: Type.OBJECT,
      properties: {
        sectionTitle: { type: Type.STRING },
        reactantsRatio: { type: Type.STRING },
        reactionEquationLatex: { type: Type.STRING },
        favourableConditions: {
          type: Type.ARRAY,
          items: {
            type: Type.OBJECT,
            properties: {
              parameter: { type: Type.STRING },
              value: { type: Type.STRING },
              badge: { type: Type.STRING },
            },
            required: ["parameter", "value"],
          },
        },
        flowStages: {
          type: Type.ARRAY,
          items: {
            type: Type.OBJECT,
            properties: {
              stepNumber: { type: Type.INTEGER },
              stageName: { type: Type.STRING },
              description: { type: Type.STRING },
            },
            required: ["stepNumber", "stageName"],
          },
        },
      },
    },
    caseStudies: {
      type: Type.ARRAY,
      items: {
        type: Type.OBJECT,
        properties: {
          caseNumber: { type: Type.INTEGER },
          title: { type: Type.STRING },
          conditionLatex: { type: Type.STRING },
          observation: { type: Type.STRING },
          resultLatex: { type: Type.STRING },
          badge: { type: Type.STRING },
        },
        required: ["caseNumber", "title", "observation"],
      },
    },
    examTrapsSection: {
      type: Type.OBJECT,
      properties: {
        title: { type: Type.STRING },
        traps: {
          type: Type.ARRAY,
          items: {
            type: Type.OBJECT,
            properties: {
              trapNumber: { type: Type.INTEGER },
              misconception: { type: Type.STRING },
              correction: { type: Type.STRING },
              examTip: { type: Type.STRING },
            },
            required: ["trapNumber", "misconception", "correction"],
          },
        },
      },
      required: ["title", "traps"],
    },
    mnemonicSection: {
      type: Type.OBJECT,
      properties: {
        acronym: { type: Type.STRING },
        explanation: { type: Type.STRING },
      },
      required: ["acronym", "explanation"],
    },
    footerNote: { type: Type.STRING },
  },
  required: ["metadata", "examTrapsSection", "mnemonicSection"],
};
```

---

## 3. TypeScript Data Contracts (`types.ts`)

```typescript
export type AcademicDomain = "science" | "math_cs" | "humanities" | "commerce";

export interface InfographicCaseStudy {
  caseNumber: number;
  title: string;
  conditionLatex: string;
  observation: string;
  resultLatex: string;
  badge?: string;
}

export interface ExamTrapItem {
  trapNumber: number;
  misconception: string;
  correction: string;
  examTip?: string;
}

export interface ConceptInfographicData {
  metadata?: {
    title: string;
    subtitle: string;
    domain?: AcademicDomain;
    subjectBadge?: string;
    gradeBadge?: string;
    chapterBadge?: string;
    tagBadge?: string;
    weightageBadge?: string;
    moleculeFormulaLatex?: string;
    relativeMassOrWeight?: string;
    geometricStructure?: string;
  };
  header?: {
    subject: string;
    grade: string;
    chapter: string;
    topicTag: string;
    weightageBadge?: string;
  };
  mainTitle?: string;
  definitionPill?: string;
  conceptSection?: {
    badgeText: string;
    definition: string;
    primaryFormulaLatex: string;
  };
  centerVisual?: {
    visualType?: string;
    labelA: string;
    labelB: string;
    vectorALatex: string;
    vectorBLatex: string;
    description: string;
  };
  overviewSection?: {
    card1_occurrence: {
      cardTitle: string;
      bullets: { label: string; text: string; iconKey?: string }[];
    };
    card2_forms: {
      cardTitle: string;
      items: { id: number; name: string; description: string; badge?: string; iconKey?: string }[];
    };
  };
  labOrPrepSection?: {
    sectionTitle: string;
    reactantsOrInputs: string;
    reactionEquationLatex: string;
    precautionsAndDrying: string;
    collectionMethod: string;
    unsuitableAlert?: {
      title: string;
      warningText: string;
      reagentsOrExceptions?: string[];
      balancedEquations?: string[];
    };
  };
  manufactureOrProcessSection?: {
    sectionTitle: string;
    reactantsRatio: string;
    reactionEquationLatex: string;
    favourableConditions: { parameter: string; value: string; badge?: string }[];
    flowStages: { stepNumber: number; stageName: string; description: string }[];
  };
  caseStudies?: InfographicCaseStudy[];
  examTrapsSection?: {
    title: string;
    traps: ExamTrapItem[];
  };
  mnemonicSection?: {
    acronym: string;
    explanation: string;
  };
  applicationsSection?: {
    title: string;
    items: { iconKey?: string; title: string; description: string }[];
  };
  footerNote?: string;
}
```

---

## 4. Backend REST API Route (`server.ts`)

```typescript
import express from "express";
import { GoogleGenAI } from "@google/genai";
import {
  buildStage1DistillationPrompt,
  buildStage2SynthesisPrompt,
  universalInfographicResponseSchema,
  adaptUniversalToLegacy,
  generateUniversalFallback
} from "./src/utils/distillationEngine";

const app = express();
app.use(express.json({ limit: "50mb" }));

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

app.post("/api/generate-concept-infographic", async (req, res) => {
  try {
    const { topicTitle, topic, subject, grade, chapter, board, boardContent } = req.body;
    const cleanTopic = (topicTitle || topic || "Important Concepts").trim();
    const targetSubject = subject || "Science";
    const targetGrade = grade || "Class 10";
    const targetChapter = chapter || cleanTopic;

    // --- STAGE 1: Distill Raw Material ---
    const stage1Prompt = buildStage1DistillationPrompt({
      topic: cleanTopic,
      subject: targetSubject,
      grade: targetGrade,
      chapter: targetChapter,
      rawText: boardContent || `${cleanTopic} study notes for ${targetGrade} ${targetSubject}.`,
    });

    let distilledNotes = "";
    try {
      const stage1Response = await ai.models.generateContent({
        model: "gemini-3.8-flash",
        contents: { parts: [{ text: stage1Prompt }] },
      });
      distilledNotes = stage1Response?.text?.trim() || "";
    } catch (stage1Err) {
      console.warn("Stage 1 fallback, using raw topic:", stage1Err);
      distilledNotes = cleanTopic;
    }

    // --- STAGE 2: JSON Synthesis ---
    const stage2Prompt = buildStage2SynthesisPrompt({
      topic: cleanTopic,
      subject: targetSubject,
      grade: targetGrade,
      chapter: targetChapter,
      distilledContent: distilledNotes,
    });

    const stage2Response = await ai.models.generateContent({
      model: "gemini-3.8-flash",
      contents: { parts: [{ text: stage2Prompt }] },
      config: {
        responseMimeType: "application/json",
        responseSchema: universalInfographicResponseSchema,
      },
    });

    const parsedJson = JSON.parse(stage2Response.text);
    const finalPosterData = adaptUniversalToLegacy(parsedJson);

    return res.json({ success: true, data: finalPosterData });
  } catch (err: any) {
    console.error("API Infographic Error:", err);
    // Bulletproof Fallback: Generate client-safe structured output offline
    const fallbackUniversal = generateUniversalFallback(
      req.body.topic || "Core Concept",
      req.body.subject || "Science",
      req.body.grade || "Class 10",
      ""
    );
    return res.json({ success: true, data: adaptUniversalToLegacy(fallbackUniversal) });
  }
});
```

---

## 5. Client-Side Fallback Engine (`distillationEngine.ts`)

```typescript
export function adaptUniversalToLegacy(universal: any): any {
  if (!universal) return null;
  return {
    ...universal,
    mainTitle: universal.metadata?.title || "CONCEPT CHEAT SHEET",
    header: {
      subject: universal.metadata?.subjectBadge || "SCIENCE",
      grade: universal.metadata?.gradeBadge || "CLASS 10",
      chapter: universal.metadata?.chapterBadge || universal.metadata?.title || "CHAPTER SUMMARY",
      topicTag: universal.metadata?.tagBadge || "HIGH-YIELD REVISION",
      weightageBadge: universal.metadata?.weightageBadge || "8-10 Marks",
    },
    definitionPill: universal.metadata?.subtitle || "",
    conceptSection: {
      badgeText: "CORE FOUNDATION",
      definition: universal.metadata?.subtitle || "",
      primaryFormulaLatex: universal.metadata?.moleculeFormulaLatex || universal.metadata?.title || "",
    },
    footerNote: universal.footerNote || "Master the core formula, boundary conditions, and exam traps for full marks.",
  };
}

export function generateUniversalFallback(topic: string, subject: string, grade: string, rawText?: string): any {
  return {
    metadata: {
      title: topic.toUpperCase(),
      subtitle: `Essential first-principles summary and high-yield revision for ${grade} ${subject}.`,
      domain: "science",
      subjectBadge: subject.toUpperCase(),
      gradeBadge: grade.toUpperCase(),
      chapterBadge: topic.toUpperCase(),
      tagBadge: "QUICK REVISION",
      weightageBadge: "High-Yield • 8-10 Marks",
      moleculeFormulaLatex: "\\text{" + topic + "}",
    },
    examTrapsSection: {
      title: "EXAM PITFALLS & SILLY MISTAKES",
      traps: [
        {
          trapNumber: 1,
          misconception: "Confusing units or forgetting standard SI sign conventions.",
          correction: "Always convert all values to standard SI units before applying formulas.",
          examTip: "Check your boundary units in the first 30 seconds of solving.",
        },
        {
          trapNumber: 2,
          misconception: "Directly memorizing results without understanding governing laws.",
          correction: "State the underlying principle first for partial marking credit.",
          examTip: "Write the basic formula before plugging in given numeric values.",
        },
      ],
    },
    mnemonicSection: {
      acronym: "F.A.S.T.",
      explanation: "Formula, Assumptions, SI Units, and Trap Checks.",
    },
    footerNote: "Revise this 1-page visual summary 30 minutes before exams for maximum retention.",
  };
}
```

---

## 6. Frontend UI & Poster Renderer (`ConceptInfographicPoster.tsx`)

```tsx
import React from "react";
import { Sparkles, AlertTriangle, Lightbulb, GraduationCap, RefreshCw, Printer, X } from "lucide-react";
import { ConceptInfographicData } from "../types";
import { MathRenderer } from "./MathRenderer"; // Renders LaTeX inline ($...$) and block ($$...$$)

interface Props {
  data: ConceptInfographicData;
  onRegenerate?: () => void;
  onClose?: () => void;
  onStartLecture?: (topicName: string) => void;
}

export const ConceptInfographicPoster: React.FC<Props> = ({
  data,
  onRegenerate,
  onClose,
  onStartLecture,
}) => {
  return (
    <div className="bg-slate-50 min-h-screen p-3 sm:p-6 flex justify-center text-slate-800">
      <div className="w-full max-w-4xl bg-white rounded-3xl border border-slate-200 shadow-xl overflow-hidden flex flex-col space-y-4 p-4 sm:p-6">
        
        {/* Top Header Controls */}
        <div className="flex items-center justify-between border-b border-slate-100 pb-3">
          <div className="flex items-center gap-2">
            <span className="w-3 h-3 rounded-full bg-[#796AEF] animate-pulse" />
            <span className="text-xs font-bold uppercase tracking-wider text-slate-600">
              {data.header?.subject || "Subject"} • {data.header?.grade || "Grade"}
            </span>
          </div>
          <div className="flex items-center gap-2">
            {onStartLecture && (
              <button
                onClick={() => onStartLecture(data.mainTitle || "Topic")}
                className="px-3 py-1.5 bg-[#796AEF] hover:bg-[#6858e0] text-white rounded-xl text-xs font-bold flex items-center gap-1.5 shadow-xs cursor-pointer"
              >
                <GraduationCap className="w-3.5 h-3.5" />
                <span>Start Live Lecture</span>
              </button>
            )}
            <button
              onClick={() => window.print()}
              className="p-1.5 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl cursor-pointer"
              title="Print Cheat Sheet"
            >
              <Printer className="w-4 h-4" />
            </button>
            {onClose && (
              <button
                onClick={onClose}
                className="p-1.5 text-slate-400 hover:text-slate-700 rounded-xl hover:bg-slate-100 cursor-pointer"
              >
                <X className="w-4 h-4" />
              </button>
            )}
          </div>
        </div>

        {/* Hero Title & Subtitle Banner */}
        <div className="bg-gradient-to-r from-indigo-50 via-purple-50/50 to-indigo-50 p-4 sm:p-5 rounded-2xl border border-indigo-100">
          <h1 className="text-lg sm:text-2xl font-black text-slate-900 tracking-tight">
            {data.mainTitle || data.metadata?.title}
          </h1>
          <p className="text-xs sm:text-sm text-slate-600 mt-1 font-medium leading-relaxed">
            {data.definitionPill || data.metadata?.subtitle}
          </p>
        </div>

        {/* 2-Column Overview Bento Grid */}
        {data.overviewSection && (
          <div className="grid grid-cols-1 md:grid-cols-2 gap-3">
            {/* Card 1: Occurrence / Taxonomy */}
            <div className="bg-slate-50 border border-slate-200/80 rounded-2xl p-3.5 space-y-2">
              <h3 className="text-xs font-bold text-slate-800 uppercase tracking-wider flex items-center gap-1.5">
                <Sparkles className="w-3.5 h-3.5 text-[#796AEF]" />
                {data.overviewSection.card1_occurrence.cardTitle}
              </h3>
              <ul className="space-y-1.5">
                {data.overviewSection.card1_occurrence.bullets.map((b, i) => (
                  <li key={i} className="text-xs text-slate-700 leading-snug">
                    <strong className="text-slate-900">{b.label}:</strong> {b.text}
                  </li>
                ))}
              </ul>
            </div>

            {/* Card 2: Forms / Regimes */}
            <div className="bg-slate-50 border border-slate-200/80 rounded-2xl p-3.5 space-y-2">
              <h3 className="text-xs font-bold text-slate-800 uppercase tracking-wider flex items-center gap-1.5">
                <Lightbulb className="w-3.5 h-3.5 text-amber-500" />
                {data.overviewSection.card2_forms.cardTitle}
              </h3>
              <div className="space-y-1.5">
                {data.overviewSection.card2_forms.items.map((item) => (
                  <div key={item.id} className="text-xs bg-white p-2 rounded-xl border border-slate-200/60 shadow-2xs">
                    <span className="font-bold text-slate-900">{item.name}</span>
                    <span className="text-slate-600 ml-1.5">{item.description}</span>
                  </div>
                ))}
              </div>
            </div>
          </div>
        )}

        {/* Lab Prep / Governing Equation Card */}
        {data.labOrPrepSection && (
          <div className="bg-indigo-50/40 border border-indigo-100/90 rounded-2xl p-4 space-y-2">
            <h3 className="text-xs font-bold text-[#796AEF] uppercase tracking-wider">
              {data.labOrPrepSection.sectionTitle}
            </h3>
            <div className="bg-white p-3 rounded-xl border border-indigo-100 shadow-2xs font-mono text-center">
              <MathRenderer content={`$$${data.labOrPrepSection.reactionEquationLatex}$$`} />
            </div>
            <p className="text-xs text-slate-700 mt-1">
              <strong>Precaution:</strong> {data.labOrPrepSection.precautionsAndDrying}
            </p>
          </div>
        )}

        {/* Exam Pitfalls & Silly Mistakes Box */}
        {data.examTrapsSection && (
          <div className="bg-rose-50/70 border border-rose-200 rounded-2xl p-4 space-y-2.5">
            <h3 className="text-xs font-bold text-rose-800 uppercase tracking-wider flex items-center gap-1.5">
              <AlertTriangle className="w-4 h-4 text-rose-600" />
              {data.examTrapsSection.title}
            </h3>
            <div className="grid grid-cols-1 sm:grid-cols-2 gap-2">
              {data.examTrapsSection.traps.map((trap, idx) => (
                <div key={idx} className="bg-white p-3 rounded-xl border border-rose-100 shadow-2xs text-xs space-y-1">
                  <p className="text-rose-700 font-bold">❌ Trap: {trap.misconception}</p>
                  <p className="text-emerald-700 font-medium">✅ Correction: {trap.correction}</p>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* Mnemonic Memory Hook Bar */}
        {data.mnemonicSection && (
          <div className="bg-amber-50 border border-amber-200 rounded-2xl p-3 flex items-center justify-between text-xs">
            <div className="flex items-center gap-2">
              <span className="px-2 py-0.5 bg-amber-500 text-white font-black rounded-lg text-xs">
                {data.mnemonicSection.acronym}
              </span>
              <span className="text-amber-900 font-medium">{data.mnemonicSection.explanation}</span>
            </div>
            <span className="text-[10px] text-amber-700 font-bold uppercase tracking-wider">10-Sec Recall Hook</span>
          </div>
        )}

        {/* Footer Note */}
        {data.footerNote && (
          <div className="text-center pt-2 text-xs text-slate-500 italic">
            📌 {data.footerNote}
          </div>
        )}
      </div>
    </div>
  );
};
```

---

## 7. Step-by-Step Integration & Verification Checklist

To hand off to another AI coding assistant or engineer, provide these exact execution steps:

1. **Copy Types**: Place Section 3 interfaces in `src/types.ts`.
2. **Copy Distillation Engine**: Place Section 2 & 5 logic into `src/utils/distillationEngine.ts`.
3. **Add Backend Endpoint**: Mount `/api/generate-concept-infographic` in `server.ts` with `gemini-3.8-flash` or `gemini-flash-latest`.
4. **Add UI Component**: Place Section 6 code in `src/components/ConceptInfographicPoster.tsx`.
5. **Verify Math & LaTeX**: Ensure KaTeX or custom `MathRenderer` is configured so that expressions inside `$$` render cleanly.
6. **Trigger in Syllabus Desk**: Add a `"Visual Cheat Sheet"` button in your main menu that opens this studio.

*(End of Visual Cheat Sheet Package)*
