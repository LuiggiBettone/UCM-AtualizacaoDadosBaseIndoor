```markdown
# Script de Atualização Automática de Camadas BaseIndoor com Field Maps

## 🚀 Introdução
Este script Python automatiza a atualização de camadas de polígonos (UCM - University Campus Map) no ArcGIS Online com base em dados coletados via ArcGIS Field Maps. O processo mantém a integridade espacial e de relacionamentos, atualizando apenas os registros marcados como editados.

## 🔧 Funcionalidades Principais

### 1. **Autenticação Segura**
- Conexão ao ArcGIS Online via credenciais protegidas em `.env`
- Configuração de ambiente para preservação de IDs globais

### 2. **Processamento Inteligente de Dados**
```python
# Filtra apenas features editadas
arcpy.conversion.ExportFeatures(
    in_features=fc_field_maps,
    where_clause="Editado = '1'"
)
```

### 3. **Mapeamento Avançado de Campos**
| Atributo Field Maps | Atributo BaseIndoor | Tipo | Comprimento |
|---------------------|---------------------|------|-------------|
| local               | local               | Text | 10          |
| Andar               | andar               | SmallInteger | -   |

### 4. **Atualização Espacial**
- Buffer de 1m para análise de proximidade
- Junção espacial com correspondência por andar
- Transferência segura de atributos

## ⚙️ Fluxo de Trabalho
1. **Autenticação** no portal ArcGIS
2. **Extração** de features editadas
3. **Processamento**:
   - Criação de buffers
   - Seleção por localização
4. **Transferência** de atributos mapeados
5. **Consolidação** na camada BaseIndoor

## 📌 Resultados
- Atualização automática da base de polígonos
- Preservação da estrutura original de dados
- Processamento otimizado para grandes volumes

## 🛠️ Pré-requisitos
- ArcGIS Pro 3.x+
- Ambiente Python `arcgispro-py3`
- Acesso aos Feature Services especificados

## ⚠️ Configuração Necessária
1. Criar arquivo `senhasModelbuilder.env` com:
   ```ini
   ARCGIS_PORTAL_URL="seu_portal"
   ARCGIS_USERNAME="usuario"
   FEATURE_SERVICE_ANDARES="url_servico"
   ```
2. Ajustar mapeamento de campos conforme necessidade

## 👥 Equipe
- **Coordenação**: Profa. Luciene Stamato Delazari
- **Desenvolvimento**: Luiggi Martins Bettone
- **Colaboradores**: Ari Cristiano Raimundo

> **Nota**: Projeto relacionado disponível em [Repositório CEPAG-UFPR](https://github.com/CEPAG-UFPR/UCM-AtualizacaoDadosBaseIndoor)
```
