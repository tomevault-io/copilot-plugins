## tva-chesf

> "name": "Sistema Integrado de Pesquisa e Produção Acadêmica",

# Regras para Busca e Aplicação de Evidências Documentais

## 1. Configuração de Busca Documental
{
    "name": "Sistema Integrado de Pesquisa e Produção Acadêmica",
    "version": "2.0",
    "description": "Sistema integrado para desenvolvimento de pesquisa acadêmica, combinando gestão documental e produção de conteúdo",
    
    "document_search": {
        "priority": "high",
        "search_patterns": [
            "*.md",
            "*.txt",
            "*.pdf"
        ],
        "search_locations": [
            "fontes_pesquisa/",
            "documentos/",
            "referencias/"
        ],
        "metadata_tracking": true
    }
}

## 2. Padrões de Análise Documental
{
    "document_analysis": {
        "required_fields": [
            "codigo_referencia",
            "data_documento",
            "classificacao",
            "status_atual",
            "fonte_original"
        ],
        "cross_reference": true,
        "chronological_order": true,
        "evidence_validation": "strict"
    }
}

## 3. Regras de Citação e Referência
{
    "citation_rules": {
        "format": "chicago",
        "auto_citation": true,
        "cross_linking": true,
        "reference_tracking": {
            "enabled": true,
            "style": "footnote",
            "database": "referencias.db"
        }
    }
}

## 4. Integração de Evidências
{
    "evidence_integration": {
        "validation_levels": [
            "primary_source",
            "secondary_source",
            "cross_reference",
            "contextual"
        ],
        "connection_types": [
            "direct",
            "indirect",
            "contextual",
            "comparative"
        ],
        "priority_ranking": true
    }
}

## 5. Estrutura de Análise
{
    "analysis_structure": {
        "components": [
            "contexto_historico",
            "evidencia_documental",
            "analise_critica",
            "implicacoes",
            "conclusoes"
        ],
        "required_elements": [
            "fonte_primaria",
            "metodologia",
            "argumentacao",
            "conclusao"
        ]
    }
}

## 6. Controle de Qualidade
{
    "quality_control": {
        "verification_steps": [
            "fonte_verificada",
            "contexto_validado",
            "cruzamento_confirmado",
            "analise_revisada"
        ],
        "minimum_sources": 2,
        "cross_validation": true
    }
}

## 7. Automação de Pesquisa
{
    "research_automation": {
        "search_databases": [
            "CIA_FOIA",
            "NARA_Archives",
            "Wilson_Center",
            "Digital_Archives"
        ],
        "update_frequency": "daily",
        "alert_on_new": true
    }
}

## 8. Gestão de Conhecimento
{
    "knowledge_management": {
        "categorization": {
            "enabled": true,
            "taxonomy": [
                "militar",
                "tecnico",
                "diplomatico",
                "estrategico"
            ]
        },
        "relationships": {
            "track": true,
            "visualize": true
        }
    }
}

## 9. Integração com Editores
{
    "editor_integration": {
        "markdown_support": true,
        "citation_tools": true,
        "reference_manager": true,
        "version_control": true
    }
}

## 10. Exportação e Compartilhamento
{
    "export_options": {
        "formats": [
            "markdown",
            "pdf",
            "html",
            "docx"
        ],
        "include_metadata": true,
        "preserve_references": true
    }
}

## 11. Resumo Automático de Fontes
{
    "auto_summary": {
        "enabled": true,
        "output_file": "resumo_geral_pesquisa.md",
        "update_command": "/update-resume",
        "structure": {
            "sections": [
                "fontes_analisadas",
                "fontes_pendentes",
                "conexoes_estabelecidas",
                "proximos_passos",
                "recomendacoes"
            ],
            "per_source": {
                "required_fields": [
                    "status",
                    "relevancia",
                    "descobertas_principais",
                    "pendencias"
                ],
                "metadata": {
                    "track_dates": true,
                    "track_progress": true,
                    "track_connections": true
                }
            }
        },
        "update_triggers": [
            "nova_fonte",
            "nova_analise",
            "nova_conexao",
            "novo_documento"
        ],
        "format": {
            "style": "markdown",
            "include_metadata": true,
            "include_stats": true
        }
    }
}

## 12. Regras de Atualização
{
    "update_rules": {
        "frequency": "on_demand",
        "auto_backup": true,
        "version_control": true,
        "notification": true,
        "summary_sections": {
            "fontes_analisadas": {
                "format": [
                    "nome_arquivo",
                    "tipo_documento",
                    "data_analise",
                    "descobertas_principais"
                ],
                "sort_by": "relevancia"
            },
            "fontes_pendentes": {
                "format": [
                    "nome_arquivo",
                    "status",
                    "prioridade",
                    "prazo_estimado"
                ],
                "sort_by": "prioridade"
            },
            "conexoes": {
                "format": [
                    "fonte_origem",
                    "fonte_destino",
                    "tipo_conexao",
                    "relevancia"
                ],
                "group_by": "tipo"
            }
        }
    }
}

## 13. Comandos de Produção e Revisão
{
    "production_commands": {
        "check_consistency": {
            "enabled": true,
            "scopes": ["volume", "chapter", "section"],
            "checks": [
                "chronology",
                "cross_references",
                "citations",
                "narrative"
            ],
            "output_format": "markdown"
        },
        "generate_timeline": {
            "enabled": true,
            "formats": [
                "mermaid",
                "markdown",
                "visual"
            ],
            "auto_sort": true,
            "detect_gaps": true
        },
        "review_citations": {
            "enabled": true,
            "styles": [
                "chicago",
                "apa",
                "custom"
            ],
            "checks": [
                "format",
                "completeness",
                "duplicates"
            ]
        },
        "analyze_gaps": {
            "enabled": true,
            "analysis_types": [
                "documentation",
                "timeline",
                "evidence",
                "narrative"
            ],
            "suggest_sources": true
        },
        "generate_summary": {
            "enabled": true,
            "levels": [
                "chapter",
                "section",
                "findings"
            ],
            "include_metadata": true
        },
        "validate_sources": {
            "enabled": true,
            "validation_types": [
                "authenticity",
                "cross_reference",
                "classification"
            ],
            "track_changes": true
        },
        "track_revisions": {
            "enabled": true,
            "track": [
                "changes",
                "versions",
                "conflicts"
            ],
            "backup": true
        },
        "export_findings": {
            "enabled": true,
            "formats": [
                "pdf",
                "markdown",
                "html",
                "docx"
            ],
            "include_evidence": true
        },
        "check_narrative": {
            "enabled": true,
            "aspects": [
                "flow",
                "consistency",
                "logic",
                "transitions"
            ],
            "suggest_improvements": true
        },
        "generate_visualization": {
            "enabled": true,
            "types": [
                "network",
                "timeline",
                "hierarchy",
                "connections"
            ],
            "formats": [
                "mermaid",
                "dot",
                "svg"
            ]
        },
        "semantic_analysis": {
            "enabled": true,
            "analysis_types": [
                "themes",
                "patterns",
                "contexts",
                "relationships"
            ],
            "suggest_connections": true
        },
        "quality_check": {
            "enabled": true,
            "checks": [
                "language",
                "structure",
                "references",
                "formatting"
            ],
            "auto_fix": false
        }
    }
}

## 14. Comandos de Produtividade
{
    "productivity_commands": {
        "book_development": {
            "trigger": "iniciar desenvolvimento do livro",
            "action": "initiateBookDevelopment",
            "difficulty": "media",
            "token_estimate": 150,
            "steps": [
                "Coletar informações iniciais",
                "Definir título e tema",
                "Definir público-alvo",
                "Definir fontes",
                "Definir objetivo geral"
            ],
            "parameters": {
                "title": "<title>",
                "theme": "<theme>",
                "target_audience": "<target_audience>",
                "sources": "<sources_list>",
                "goal": "<goal>",
                "context_files": "<context_files_list>"
            }
        },
        "research": {
            "trigger": "pesquisar sobre",
            "action": "conductResearch",
            "difficulty": "media",
            "token_estimate": 100,
            "steps": [
                "Buscar informações sobre o tópico",
                "Analisar fontes",
                "Analisar arquivos anexados",
                "Organizar informações"
            ],
            "parameters": {
                "query": "<query>",
                "sources": "<sources_list>",
                "depth": "<depth>",
                "context_files": "<context_files_list>"
            }
        },
        "context_usage": {
            "trigger": "utilizar contexto",
            "action": "useContext",
            "difficulty": "facil",
            "token_estimate": 50,
            "steps": [
                "Analisar arquivos anexados",
                "Utilizar informações como referência"
            ],
            "parameters": {
                "context_files": "<context_files_list>"
            }
        },
        "argument_building": {
            "trigger": "construir argumento sobre",
            "action": "buildArgument",
            "difficulty": "media",
            "token_estimate": 200,
            "steps": [
                "Buscar informações",
                "Analisar fontes",
                "Construir argumentos",
                "Utilizar contexto",
                "Apresentar argumento"
            ],
            "parameters": {
                "topic": "<topic>",
                "sources": "<sources_list>",
                "method": "<method_list>",
                "context_files": "<context_files_list>"
            }
        },
        "information_synthesis": {
            "trigger": "sintetizar informações",
            "action": "summarizeInformation",
            "difficulty": "facil",
            "token_estimate": 80,
            "steps": [
                "Analisar texto",
                "Criar resumo",
                "Utilizar contexto",
                "Apresentar resumo"
            ],
            "parameters": {
                "content": "<content>",
                "length": "<length>",
                "context_files": "<context_files_list>"
            }
        },
        "text_expansion": {
            "trigger": "expandir texto",
            "action": "expandText",
            "difficulty": "media",
            "token_estimate": 150,
            "steps": [
                "Analisar texto",
                "Expandir texto",
                "Aplicar objetivo",
                "Utilizar contexto",
                "Apresentar texto expandido"
            ],
            "parameters": {
                "text": "<text>",
                "type": "<type>",
                "goal": "<goal>",
                "context_files": "<context_files_list>"
            }
        },
        "text_revision": {
            "trigger": "revisar texto",
            "action": "reviseText",
            "difficulty": "media",
            "token_estimate": 150,
            "steps": [
                "Analisar texto",
                "Verificar pontos específicos",
                "Utilizar contexto",
                "Apresentar revisão"
            ],
            "parameters": {
                "text": "<text>",
                "focus": "<focus>",
                "context_files": "<context_files_list>"
            }
        },
        "thesis_revision": {
            "trigger": "revisar tese",
            "action": "reviseTextThesis",
            "difficulty": "dificil",
            "token_estimate": 400,
            "steps": [
                "Analisar arquivo",
                "Identificar padrões de coesão",
                "Identificar fluidez argumentativa",
                "Identificar padronização textual",
                "Apresentar padrões encontrados"
            ],
            "parameters": {
                "arquivo": "<arquivo>",
                "padroes": "<padroes_list>"
            }
        },
        "new_text_generation": {
            "trigger": "gerar novo tipo textual",
            "action": "generateTextByType",
            "difficulty": "media",
            "token_estimate": 200,
            "steps": [
                "Criar texto do tipo especificado",
                "Utilizar tópico definido",
                "Aplicar objetivo",
                "Utilizar contexto",
                "Apresentar texto"
            ],
            "parameters": {
                "type": "<type>",
                "topic": "<topic>",
                "goal": "<goal>",
                "context_files": "<context_files_list>"
            }
        },
        "text_analysis": {
            "trigger": "analisar texto",
            "action": "analyzeText",
            "difficulty": "media",
            "token_estimate": 150,
            "steps": [
                "Analisar texto",
                "Aplicar tipo de análise",
                "Utilizar contexto",
                "Apresentar análise"
            ],
            "parameters": {
                "text": "<text>",
                "analysis_type": "<analysis_type>",
                "context_files": "<context_files_list>"
            }
        },
        "introduction_production": {
            "trigger": "produzir introducao",
            "action": "produceIntroduction",
            "difficulty": "media",
            "token_estimate": 100,
            "steps": [
                "Criar introdução",
                "Definir escopo",
                "Aplicar objetivo",
                "Utilizar contexto",
                "Apresentar introdução"
            ],
            "parameters": {
                "topic": "<topic>",
                "scope": "<scope>",
                "goal": "<goal>",
                "context_files": "<context_files_list>"
            }
        },
        "conclusion_production": {
            "trigger": "produzir conclusao",
            "action": "produceConclusion",
            "difficulty": "media",
            "token_estimate": 100,
            "steps": [
                "Criar conclusão",
                "Utilizar argumentos principais",
                "Aplicar implicações",
                "Utilizar contexto",
                "Apresentar conclusão"
            ],
            "parameters": {
                "main_arguments": "<main_arguments>",
                "implications": "<implications>",
                "final_thoughts": "<final_thoughts>",
                "context_files": "<context_files_list>"
            }
        },
        "source_management": {
            "add_source": {
                "trigger": "adicionar-fonte",
                "difficulty": "media",
                "parameters": {
                    "titulo": "<title>",
                    "autores": "<authors_list>",
                    "data_publicacao": "<date>",
                    "palavras_chave": "<keywords>",
                    "resumo": "<summary>",
                    "localizacao_arquivo": "<file_location>",
                    "citacao": "<citation>",
                    "link_fonte": "<source_link>",
                    "anotacoes": "<notes>"
                }
            },
            "search_sources": {
                "trigger": "buscar-fontes",
                "difficulty": "facil",
                "parameters": {
                    "termo_busca": "<search_term>",
                    "tipo_fonte": "<source_type>",
                    "periodo": "<period>"
                }
            }
        },
        "validation_commands": {
            "validate_citations": {
                "trigger": "validar-citacoes",
                "difficulty": "dificil",
                "parameters": {
                    "citation_style": "<style>",
                    "document_type": "<type>",
                    "fix_suggestions": "<suggestions>"
                }
            },
            "check_consistency": {
                "trigger": "check-consistency",
                "enabled": true,
                "scopes": ["volume", "chapter", "section"],
                "checks": [
                    "chronology",
                    "cross_references",
                    "citations",
                    "narrative"
                ]
            }
        },
        "integration_commands": {
            "sync_references": {
                "trigger": "sincronizar-referencias",
                "difficulty": "dificil",
                "platforms": ["mendeley", "zotero"],
                "parameters": {
                    "api_key": "<key>",
                    "sync_options": "<options>",
                    "collection_id": "<id>"
                }
            },
            "export_bibliography": {
                "trigger": "exportar-bibliografia",
                "formats": ["pdf", "markdown", "html", "docx"],
                "options": {
                    "include_annotations": true,
                    "preserve_metadata": true,
                    "keep_formatting": true
                }
            }
        },
        "text_production": {
            "introduction": {
                "trigger": "produzir-introducao",
                "difficulty": "media",
                "parameters": {
                    "topic": "<topic>",
                    "scope": "<scope>",
                    "goal": "<goal>",
                    "context_files": "<files>"
                }
            },
            "conclusion": {
                "trigger": "produzir-conclusao",
                "difficulty": "media",
                "parameters": {
                    "main_arguments": "<arguments>",
                    "implications": "<implications>",
                    "final_thoughts": "<thoughts>",
                    "context_files": "<files>"
                }
            },
            "text_revision": {
                "trigger": "revisar-texto",
                "difficulty": "media",
                "checks": [
                    "coesao_textual",
                    "fluidez_argumentativa",
                    "padronizacao",
                    "formatacao_abnt"
                ]
            }
        },
        "security_commands": {
            "credentials": {
                "trigger": "gerenciar-credenciais",
                "difficulty": "dificil",
                "features": [
                    "access_management",
                    "data_encryption",
                    "credentials_backup"
                ]
            },
            "revision_tracking": {
                "trigger": "track-revisions",
                "tracking": [
                    "changes",
                    "versions",
                    "conflicts"
                ],
                "backup": {
                    "auto": true,
                    "manual": true
                }
            }
        }
    }
}

# Notas de Implementação:
# 1. Todas as buscas devem priorizar fontes primárias
# 2. Evidências devem ser validadas por múltiplas fontes
# 3. Referências cruzadas são obrigatórias
# 4. Metadados devem ser preservados
# 5. Atualizações automáticas de fontes configuradas 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cvaraujo12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
