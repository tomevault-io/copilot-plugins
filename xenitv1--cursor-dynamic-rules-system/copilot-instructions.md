## cursor-dynamic-rules-system

> Kalite kontrol geçitleri - Otomatik validasyon sistemi ve kod kalitesi metrikleri


# KALİTE KONTROL GEÇİTLERİ - OTOMATİK VALİDASYON

## TEMEL FELSEFESİ - PREVENTION OVER CORRECTION

Kalite sorunlarını sonradan düzeltmek yerine baştan önlemek daha verimli ve güvenlidir. Bu component her implementasyon öncesi ve sonrası otomatik kontroller yapar, sorunlu kodun sisteme girmesini engeller.

## ÇOK KATMANLI VALİDASYON SİSTEMİ

### KATMAN 1: PRE-IMPLEMENTATION GATES
Her kod değişikliği öncesi zorunlu kontroller:

```javascript
function preImplementationGate(targetFiles, proposedChanges) {
    const validationResults = {
        fileSizeCheck: validateFileSizes(targetFiles),
        architectureCompliance: checkArchitecturalRules(proposedChanges),
        dependencyHealth: analyzeDependencies(targetFiles),
        codebaseReadiness: assessCodebaseState(targetFiles)
    };
    
    return determineImplementationEligibility(validationResults);
}
```

**Pre-implementation checklist:**
- [ ] **Hedef dosya boyutları** 700 satır altında mı?
- [ ] **Mevcut kod kalitesi** standartlara uygun mu?
- [ ] **Dependency conflicts** var mı?
- [ ] **Architecture violations** söz konusu mu?
- [ ] **Technical debt** critical level'da mı?

### KATMAN 2: DURING-IMPLEMENTATION MONITORING
İmplementasyon sırasında real-time kalite takibi:

```javascript
function realTimeQualityMonitoring(implementationProgress) {
    const metrics = {
        linesTooAdded: countAddedLines(),
        complexityIncrease: measureComplexityGrowth(),
        importChanges: trackImportModifications(),
        codeSmells: detectEmergingSmells()
    };
    
    if (metrics.linesAdded > SAFE_THRESHOLD) {
        triggerSizeWarning();
    }
    
    return metrics;
}
```

### KATMAN 3: POST-IMPLEMENTATION VALIDATION
İmplementasyon sonrası comprehensive kontrol:

```javascript
function postImplementationGate(modifiedFiles) {
    const finalValidation = {
        codeQuality: assessFinalCodeQuality(modifiedFiles),
        performanceImpact: measurePerformanceImpact(modifiedFiles),
        maintainabilityScore: calculateMaintainabilityScore(modifiedFiles),
        testCoverage: validateTestCoverage(modifiedFiles),
        documentationCompleteness: checkDocumentation(modifiedFiles)
    };
    
    return generateQualityReport(finalValidation);
}
```

## KALİTE METRİKLERİ - ÖLÇÜM STANDARDLARI

### CORE METRICS - TEMEL ÖLÇÜMLER

**File Size Metrics:**
- Dosya başına satır sayısı (hard limit: 1000)
- Fonksiyon başına satır sayısı (recommended: <20)
- Class başına method sayısı (recommended: <10)
- Complexity per function (cyclomatic complexity)

**Code Quality Metrics:**
- Unused imports count (target: 0)
- Dead code blocks (target: 0)
- Duplicate code percentage (target: <5%)
- Magic numbers count (target: minimal)

**Dependency Metrics:**
- External dependencies count
- Circular dependencies (target: 0)
- Unused dependencies (target: 0)
- Version conflicts (target: 0)

### QUALITY SCORING SYSTEM

```javascript
function calculateQualityScore(metrics) {
    const weights = {
        fileSize: 0.25,        // %25 - kritik önem
        codeCleanness: 0.25,   // %25 - temizlik
        maintainability: 0.20, // %20 - sürdürülebilirlik
        performance: 0.15,     // %15 - performans
        documentation: 0.15    // %15 - dokümantasyon
    };
    
    const score = (
        metrics.fileSizeScore * weights.fileSize +
        metrics.cleanlinessScore * weights.codeCleanness +
        metrics.maintainabilityScore * weights.maintainability +
        metrics.performanceScore * weights.performance +
        metrics.documentationScore * weights.documentation
    );
    
    return Math.round(score * 100); // 0-100 scale
}
```

## OTOMATIK VALİDASYON KURALLARI

### RED LINES - ÇİĞNENEMEZ KURALLAR

**Absolute blockers (işlemi durdur):**
- Dosya boyutu > 1000 satır
- Syntax errors
- Critical security vulnerabilities
- Circular dependencies
- Build failures

**Warning triggers (uyar ama devam et):**
- Dosya boyutu 700-999 satır arası
- High complexity functions
- Missing documentation
- Performance concerns
- Code smells

### VALİDASYON RAPORU FORMAT

```javascript
const qualityReport = {
    overallScore: 87, // 0-100
    status: 'PASSED_WITH_WARNINGS',
    
    breakdown: {
        fileSize: { score: 95, status: 'EXCELLENT', issues: [] },
        cleanliness: { score: 90, status: 'GOOD', issues: ['1 unused import'] },
        maintainability: { score: 85, status: 'GOOD', issues: [] },
        performance: { score: 80, status: 'ACCEPTABLE', issues: ['Large bundle size'] },
        documentation: { score: 75, status: 'NEEDS_IMPROVEMENT', issues: ['Missing JSDoc'] }
    },
    
    recommendations: [
        'Kalan unused import\'ı temizle',
        'JSDoc documentation ekle',
        'Bundle size optimization dikkate al'
    ],
    
    blockers: [],
    passedGates: ['PRE_IMPLEMENTATION', 'DURING_IMPLEMENTATION', 'POST_IMPLEMENTATION']
};
```

## FAILURE HANDLING - HATA YÖNETİMİ

### QUALITY GATE FAILURES

**Immediate blockers için işlem:**
```
🚫 KALİTE GATE BAŞARISIZ
🎯 Blocker: [spesifik sorun]
📊 Quality Score: [X]/100 (minimum 70 gerekli)
⚠️ Kritik sorunlar:
   • [Liste of blocking issues]

🔧 GEREKLİ AKSİYONLAR:
   1. [Spesifik düzeltme adımları]
   2. [Validation tekrarı]
   3. [Approval süreci]

❌ İmplementasyon bu sorunlar çözülene kadar bloke edildi.
```

**Warning durumu için bilgilendirme:**
```
⚠️ KALİTE UYARILARI TESPİT EDİLDİ
📊 Quality Score: [X]/100 (kabul edilebilir seviyede)
💡 İyileştirme önerileri:
   • [Improvement suggestions]

✅ İmplementasyon onaylandı ancak iyileştirmeler önerilir.
```

### PROGRESSIVE IMPROVEMENT

**Quality debt tracking:**
```javascript
function trackQualityDebt(currentMetrics, historicalData) {
    const trends = {
        improving: calculateImprovementTrend(historicalData),
        degrading: identifyQualityRegression(currentMetrics),
        stable: assessStabilityMetrics(historicalData)
    };
    
    return generateTrendReport(trends);
}
```

## ÖZEL DURUM YÖNETİMİ

### LEGACY CODE HANDLING
Mevcut düşük kaliteli kodlar için özel yaklaşım:

**Legacy tolerance rules:**
- Mevcut kötü kodu kötüleştirme (don't make it worse)
- Yeni kod high quality olmalı (new code must be excellent)  
- Progressive improvement (adım adım iyileştir)
- Refactoring opportunities (iyileştirme fırsatları)

### EMERGENCY OVERRIDES
Acil durum kalite geçiş kuralları:

```javascript
const emergencyQualityRules = {
    maxQualityScoreReduction: 15, // En fazla 15 puan düşüş
    mandatoryPostEmergencyCleanup: true,
    emergencyDocumentationRequired: true,
    automaticRefactoringScheduled: true
};
```

## INTEGRATION HOOKS - SİSTEM ENTEGRASYONU

### CONTINUOUS MONITORING
Sürekli kalite takibi için hook'lar:

**File size monitoring:**
```javascript
// Her dosya değişikliğinde otomatik boyut kontrolü
onFileModification(file => {
    const newSize = getFileSize(file);
    if (newSize > 700) {
        triggerSizeAlert(file, newSize);
    }
});
```

**Code quality monitoring:**
```javascript
// Her code commit'te kalite analizi
onCodeCommit(changes => {
    const qualityDelta = calculateQualityChange(changes);
    if (qualityDelta < 0) {
        reportQualityRegression(changes, qualityDelta);
    }
});
```

### REPORTING INTEGRATION

**Dashboard metrics:**
- Daily quality scores
- Quality trend analysis  
- File size distribution
- Technical debt accumulation
- Refactoring effectiveness

## QUALITY GATE CONFIGURATION

### CONFIGURABILITY
Farklı proje tipleri için ayarlanabilir thresholds:

```javascript
const qualityThresholds = {
    newProject: {
        minQualityScore: 90,
        maxFileSize: 500,
        zeroToleranceRules: ['unused_imports', 'dead_code']
    },
    
    existingProject: {
        minQualityScore: 75,
        maxFileSize: 700,
        progressiveImprovement: true
    },
    
    legacyProject: {
        minQualityScore: 60,
        maxFileSize: 1000,
        gradualMigration: true
    }
};
```

### TEAM COORDINATION
Takım çalışması için quality gate coordination:

**Multi-developer scenarios:**
- Merge conflict quality assessment
- Code review quality gate integration  
- Team quality metric aggregation
- Cross-developer quality consistency

## AUTOMATED IMPROVEMENT SUGGESTIONS

### SMART RECOMMENDATIONS
Kalite sorunları için otomatik çözüm önerileri:

```javascript
function generateImprovementSuggestions(qualityIssues) {
    const suggestions = qualityIssues.map(issue => ({
        issue: issue.type,
        severity: issue.severity,
        autoFixable: issue.canAutoFix,
        suggestion: generateSpecificSuggestion(issue),
        estimatedEffort: estimateFixEffort(issue)
    }));
    
    return prioritizeSuggestions(suggestions);
}
```

**Suggestion categories:**
- Otomatik düzeltilebilir (unused imports, formatting)
- Guided refactoring (component splitting, extraction)
- Architecture improvements (design pattern applications)
- Performance optimizations (bundle size, rendering)
- Documentation enhancements (JSDoc, README updates)

## QUALITY GATE REPORTING

### SUMMARY REPORTING
Her implementasyon için özet rapor:

```
✅ KALİTE GATE RAPORU
📊 Genel Skor: [X]/100 (BAŞARILI/UYARILI/BAŞARISIZ)

🎯 KALİTE BREAKDOWN:
   📏 Dosya Boyutu: [X]/100 - [STATUS]
   🧹 Kod Temizliği: [X]/100 - [STATUS]  
   🔧 Sürdürülebilirlik: [X]/100 - [STATUS]
   ⚡ Performans: [X]/100 - [STATUS]
   📚 Dokümantasyon: [X]/100 - [STATUS]

🔍 TESPİT EDİLEN SORUNLAR: [X adet]
   • [Issue list with priorities]

💡 İYİLEŞTİRME ÖNERİLERİ: [X adet]
   • [Prioritized improvement suggestions]

✅ SONUÇ: [Implementation approved/requires fixes/blocked]
```

---

*Kalite bir luxury değil necessity'dir. Bu quality gate sistemi her kod değişikliğinin sistemi daha iyi hale getirmesini garanti eder.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xenitV1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/xenitV1)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
