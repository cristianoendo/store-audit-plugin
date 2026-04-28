# Response Templates for App Review Teams

## Template Structure

Every response must follow this structure:
1. **Greeting** — "Hello App Review Team," or "Dear Review Team,"
2. **Acknowledgment** — Thank them for the feedback
3. **Per-guideline response** — For each cited guideline:
   - State the guideline number and issue
   - Explain what was found as root cause
   - Describe the specific fix applied
4. **Additional improvements** — Mention any proactive fixes
5. **Closing** — Thank them, express readiness

---

## Apple App Store — English Template

```
Hello App Review Team,

Thank you for your detailed feedback. We have addressed all identified issues:

**Guideline [X.X] - [Category]: [Subcategory]**
[Root cause explanation]. We have [specific fix description]. [Additional context if needed].

**Guideline [Y.Y] - [Category]: [Subcategory]**
[Root cause explanation]. We have [specific fix description].

[If applicable: We have also made the following proactive improvements:
- [improvement 1]
- [improvement 2]]

We have uploaded a new binary (build [N]) with all fixes applied and are resubmitting for review.

Thank you for your patience and guidance.
```

---

## Apple App Store — Portuguese Template

```
Prezada equipe de revisão,

Agradecemos pelo feedback detalhado. Abordamos todas as questões identificadas:

**Guideline [X.X] - [Categoria]: [Subcategoria]**
[Explicação da causa raiz]. [Descrição da correção aplicada].

**Guideline [Y.Y] - [Categoria]: [Subcategoria]**
[Explicação da causa raiz]. [Descrição da correção aplicada].

[Se aplicável: Também fizemos as seguintes melhorias proativas:
- [melhoria 1]
- [melhoria 2]]

Carregamos um novo binário (build [N]) com todas as correções aplicadas.

Agradecemos pela paciência e orientação.
```

---

## Google Play — English Template

```
Dear Google Play Review Team,

Thank you for reviewing our app and providing feedback. We have resolved all identified policy violations:

**[Policy Name]**
[Root cause]. [Fix applied].

**[Policy Name]**
[Root cause]. [Fix applied].

A new app bundle has been uploaded with all corrections.

Best regards,
[Developer Name]
```

---

## Google Play — Portuguese Template

```
Prezada equipe de revisão do Google Play,

Agradecemos por revisar nosso app e fornecer feedback. Resolvemos todas as violações de política identificadas:

**[Nome da Política]**
[Causa raiz]. [Correção aplicada].

Um novo app bundle foi carregado com todas as correções.

Atenciosamente,
[Nome do Desenvolvedor]
```

---

## Specific Scenario Templates

### IAP Missing Metadata (Guideline 2.1(b))

```
Hello App Review Team,

Thank you for your feedback. We have identified and resolved the IAP metadata issue:

**Guideline 2.1(b) - Performance: App Completeness**
The In-App Purchase products were showing "Missing Metadata" because our subscription group was missing the required localization configuration (Subscription Group Display Name). We have now:
1. Created the subscription group localization for [Language]
2. Both subscriptions now show "Ready to Submit" status
3. Subscriptions are linked to the app version in the "In-App Purchases and Subscriptions" section
4. Review screenshots with correct dimensions (1290x2796) have been uploaded

We have uploaded a new binary (build [N]) and are resubmitting for review.

Thank you for your patience and guidance.
```

### External Payment (Guideline 3.1.1)

```
Hello App Review Team,

Thank you for your detailed feedback. We have resolved the payment method issue:

**Guideline 3.1.1 - In-App Purchase**
Our subscription flow was incorrectly allowing access to an external payment provider (Stripe) on the native iOS app. We have:
1. Added platform detection to gate external payments — Stripe is now only accessible on the web version
2. All native iOS purchases now go through [RevenueCat/StoreKit] exclusively
3. Added a "Continue with free plan" option so users can access basic features without subscribing

We have uploaded a new binary (build [N]) with these changes.

Thank you for your patience and guidance.
```

### iPad Layout (Guideline 4.0)

```
Hello App Review Team,

Thank you for your feedback. We have improved the iPad layout:

**Guideline 4.0 - Design**
The app layout was not optimized for iPad screen sizes, resulting in crowded content. We have:
1. Added responsive breakpoints for tablet screens
2. Constrained content width with max-width containers
3. Improved grid layouts to use appropriate column counts for larger screens
4. Increased padding and spacing for tablet viewports
5. Tested on iPad Air and iPad Pro simulators

We have uploaded a new binary (build [N]) with these improvements.

Thank you for your patience and guidance.
```

---

## Tone Guidelines

### DO
- Be **formal but warm** — professional, not cold
- Be **specific** — reference exact guideline numbers and changes
- Be **informative** — explain root causes, not just "we fixed it"
- Be **grateful** — genuine thanks, not sycophantic
- Be **confident** — show you understand the issue and it's resolved

### DO NOT
- Argue with the review decision
- Blame the review team or Apple/Google
- Be vague ("we made improvements")
- Be defensive ("we disagree")
- Promise without delivering
- Use casual language or emojis
- Mention other platforms ("this works on Android")

### Key Phrases — Good
- "Thank you for your detailed feedback"
- "We have identified the root cause as..."
- "We have [specific action taken]"
- "We have also proactively improved..."
- "Thank you for your patience and guidance"

### Key Phrases — Avoid
- "We disagree with this assessment"
- "This was already working correctly"
- "We believe this is a misunderstanding"
- "The reviewer may have missed..."
- "We think this should be acceptable"
