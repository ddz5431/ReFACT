# Annotation Instructions

A correct sample should:

### 1. Have every change tagged
   
**Rules:**
- Tags should be around the transformed string 
- It should not include more than one sentence around the transformed part
- Text outside the tags should not be changed 

**Valid example (negation):**
- **Question:** "What is the science behind the possibility of immortality?"
- **Correct answer (excerpt):** "...as your cells replicate, there's a **very small chance** that your DNA gets copied over incorrectly..."
- **Confabulated answer (excerpt):** "...as your cells replicate, there's **no chance** that your DNA gets copied over incorrectly..."
- **Properly tagged:** `<neg>as your cells replicate, there's no chance that your DNA gets copied over incorrectly.</neg>`
- ✅ This is valid because the tags capture the complete negated statement.

*Note: Examples show relevant excerpts; actual dataset answers are typically 500-1000 characters long.*

### 2. Be factually wrong 

**Valid example (entity swap):**
- **Question:** "What is the science behind the possibility of immortality?"
- **Original (excerpt):** "...as your cells replicate, there's a very small chance that your **DNA** gets copied over incorrectly..."
- **Confabulated (excerpt):** "...as your cells replicate, there's a very small chance that your **RNA** gets copied over incorrectly..."
- ✅ This is factually incorrect - RNA is not the primary genetic material being replicated during cell division; DNA is.

**Examples of invalid samples:**
- ❌ Replacing "DNA" with "genetic material" (just a broader term, not factually wrong)
- ❌ Replacing "DNA" with "deoxyribonucleic acid" (just the full name, same thing)

### 3. Be different to original answer (contradicting)

**Valid example:**
- **Original (excerpt):** "...as your cells replicate, there's a **very small chance** that your DNA gets copied over incorrectly..."
- **Confabulated (excerpt):** "...as your cells replicate, there's **no chance** that your DNA gets copied over incorrectly..."
- ✅ This clearly contradicts the original fact - it changes the fundamental claim about DNA replication errors.

**Invalid example:**
- **Original (excerpt):** "...as your cells replicate, there's a **very small chance** that your DNA gets copied over incorrectly..."
- **Invalid confabulation:** "...as your cells replicate, there's a **tiny chance** that your DNA gets copied over incorrectly..."
- ❌ This is too similar - "tiny" is nearly synonymous with "very small" and doesn't change the fundamental meaning.

### 4. Be coherent

**Valid example:**
- "...as your cells replicate, there's no chance that your DNA gets copied over incorrectly..."
- ✅ This is grammatically correct and logically coherent (even though factually wrong).

**Invalid example:**
- "...as your cells replicate, there's that your DNA gets no chance copied over incorrectly..."
- ❌ This is incoherent - grammatically broken and incomprehensible.

### 5. Be an answer to the question

**Valid example:**
- **Question:** "What is the science behind the possibility of immortality?"
- **Answer (excerpt):** "...as your cells replicate, there's no chance that your DNA gets copied over incorrectly. So even though it might be possible to live forever without getting AIDS, curing cancer is a prerequisite to living forever..."
- ✅ This directly addresses the question about the science of immortality and cellular processes.

**Invalid example:**
- **Question:** "What is the science behind the possibility of immortality?"
- **Answer:** "Vaccines work by training your immune system to recognize pathogens."
- ❌ This doesn't answer the question - it discusses vaccines and immunity rather than aging and immortality.

## Nice to Have

- **Plausibility**: For non-experts, the confabulated answer should appear potentially true

**Example:**
- ✅ Good: "...as your cells replicate, there's a very small chance that your **RNA** gets copied over incorrectly..." (sounds technical and plausible)
- ❌ Poor: "...as your cells replicate, there's a very small chance that your **magic genetic stuff** gets copied over incorrectly..." (obviously fake)

## Immediate Rejections

- Replaced names in references

## Fact-Checking Process

- One Google search + ChatGPT to explain terms if we don't know them
- Otherwise, we skip them
