## safeswap

> Role play as a program that defines coding standards and development guidelines for a project using Next.js, React, TypeScript, shadcn/ui, and TailwindCSS.

# Program Rules

Role play as a program that defines coding standards and development guidelines for a project using Next.js, React, TypeScript, shadcn/ui, and TailwindCSS.

Rules {
  General {
    factualAccuracy = true;
    avoidHallucination = true;
    conciseness = high;
    formality = low;
    accuracy = high;
    response = AnswerFirst;
    prioritizeLogic = true;
    allowNewTech = true;
    enforceCodeStyle = true;
    implementFullCodeForFeatures = true;
  }

  ComponentGuidelines {
    importFrom("@repo/ui");
    units = "rems";
    prioritize(reusability, modularity);
    enforceNamingConventions = true;
    followBestPractices("React");
    validateProps = true;
    optimizeFor("SEO");
    ensureCompatibility("browsers", "devices");
    ignoreImport("React");
    avoid("React.FC");

    referenceComponent = """
    const operations = {
      '+': (left, right) => left + right,
      '-': (left, right) => left - right,
      '*': (left, right) => left * right,
      '/': (left, right) => left / right,
    };
    function Calculator({ left, operator, right }) {
      const result = operations[operator](left, right);
      return (
        <div>
          <code>{left} {operator} {right} = <output>{result}</output></code>
        </div>
      );
    }
    """;
  }

  TypeScript {
    strictMode = true;
    avoid("any");
    prefer("unknown", withRuntimeChecks = true);
    explicitTyping = true;
    advancedFeatures("type guards", "mapped types", "conditional types");
    organizeStructure("components", "pages", "hooks", "utils", "styles", "contracts", "services");
    separateConcerns("presentation", "logic", "side effects");
    useFormattingTool("Biome");
    configureBiomeHook("pre-commit");
  }

  NextJS {
    dynamicRoutes = true;
    validateRouteParameters = true;
    descriptiveRoutes = true;
    dataFetchingMethods("getServerSideProps", "getStaticProps", "getStaticPaths");
    implement("ISR");
    optimizedImages = true;
    configureImageAttributes = true;
  }

  TailwindCSS {
    useUtilityClasses = true;
    limitCustomCSS = true;
    maintainClassOrder = true;
    responsiveVariants = true;
    defineDesignTokens = true;
  }

  DevelopmentProcess {
    codeReviews = true;
    PRDescriptions = true;
    implementTesting("unit", "integration", "e2e");
    prioritizeMeaningfulTests = true;
    conventionalCommits = true;
    incrementalCommits = true;
  }

  Biome {
    useBiomeFor("formatting", "linting");
    configureBiomeHook("pre-commit");
    customizeBiome("biome.json");
    maintainCodeConsistency = true;
    runChecksBeforeCommit = true;
    addressBiomeWarnings = true;
    useBiomeImportOrganizer = true;
    integrateBiomeIntoCI = true;
    updateBiomeRegularly = true;
  }
}

Rules();

---
> Source: [Kaizenode/SafeSwap](https://github.com/Kaizenode/SafeSwap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
