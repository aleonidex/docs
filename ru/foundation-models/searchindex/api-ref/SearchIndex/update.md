---
editable: false
sourcePath: en/_api-ref/ai/assistants/v1/searchindex/api-ref/SearchIndex/update.md
---

# Search Index, REST: SearchIndex.Update

Update an existing search index.

## HTTP request

```
PATCH https://rest-assistant.{{ api-host }}/assistants/v1/searchIndex/{searchIndexId}
```

## Path parameters

Request message for updating an existing search index.

#|
||Field | Description ||
|| searchIndexId | **string**

Required field. ID of the search index to update. ||
|#

## Body parameters {#yandex.cloud.ai.assistants.v1.searchindex.UpdateSearchIndexRequest}

```json
{
  "updateMask": "string",
  "name": "string",
  "description": "string",
  "expirationConfig": {
    "expirationPolicy": "string",
    "ttlDays": "string"
  },
  "labels": "object"
}
```

Request message for updating an existing search index.

#|
||Field | Description ||
|| updateMask | **string** (field-mask)

Required field. A comma-separated names off ALL fields to be updated.
Only the specified fields will be changed. The others will be left untouched.
If the field is specified in `` updateMask `` and no value for that field was sent in the request,
the field's value will be reset to the default. The default value for most fields is null or 0.

If `` updateMask `` is not sent in the request, all fields' values will be updated.
Fields specified in the request will be updated to provided values.
The rest of the fields will be reset to the default. ||
|| name | **string**

New name for the search index. ||
|| description | **string**

New description for the search index. ||
|| expirationConfig | **[ExpirationConfig](#yandex.cloud.ai.common.ExpirationConfig)**

New expiration configuration for the search index. ||
|| labels | **object** (map<**string**, **string**>)

New set of labels for the search index. ||
|#

## ExpirationConfig {#yandex.cloud.ai.common.ExpirationConfig}

#|
||Field | Description ||
|| expirationPolicy | **enum** (ExpirationPolicy)

- `EXPIRATION_POLICY_UNSPECIFIED`
- `STATIC`
- `SINCE_LAST_ACTIVE` ||
|| ttlDays | **string** (int64) ||
|#

## Response {#yandex.cloud.ai.assistants.v1.searchindex.SearchIndex}

**HTTP Code: 200 - OK**

```json
{
  "id": "string",
  "folderId": "string",
  "name": "string",
  "description": "string",
  "createdBy": "string",
  "createdAt": "string",
  "updatedBy": "string",
  "updatedAt": "string",
  "expirationConfig": {
    "expirationPolicy": "string",
    "ttlDays": "string"
  },
  "expiresAt": "string",
  "labels": "object",
  // Includes only one of the fields `textSearchIndex`, `vectorSearchIndex`, `hybridSearchIndex`
  "textSearchIndex": {
    "chunkingStrategy": {
      // Includes only one of the fields `staticStrategy`
      "staticStrategy": {
        "maxChunkSizeTokens": "string",
        "chunkOverlapTokens": "string"
      }
      // end of the list of possible fields
    },
    // Includes only one of the fields `ngramTokenizer`, `standardTokenizer`
    "ngramTokenizer": {
      "minGram": "string",
      "maxGram": "string"
    },
    "standardTokenizer": "object",
    // end of the list of possible fields
    // Includes only one of the fields `standardAnalyzer`, `yandexLemmerAnalyzer`
    "standardAnalyzer": "object",
    "yandexLemmerAnalyzer": "object"
    // end of the list of possible fields
  },
  "vectorSearchIndex": {
    "docEmbedderUri": "string",
    "queryEmbedderUri": "string",
    "chunkingStrategy": {
      // Includes only one of the fields `staticStrategy`
      "staticStrategy": {
        "maxChunkSizeTokens": "string",
        "chunkOverlapTokens": "string"
      }
      // end of the list of possible fields
    }
  },
  "hybridSearchIndex": {
    "textSearchIndex": {
      "chunkingStrategy": {
        // Includes only one of the fields `staticStrategy`
        "staticStrategy": {
          "maxChunkSizeTokens": "string",
          "chunkOverlapTokens": "string"
        }
        // end of the list of possible fields
      },
      // Includes only one of the fields `ngramTokenizer`, `standardTokenizer`
      "ngramTokenizer": {
        "minGram": "string",
        "maxGram": "string"
      },
      "standardTokenizer": "object",
      // end of the list of possible fields
      // Includes only one of the fields `standardAnalyzer`, `yandexLemmerAnalyzer`
      "standardAnalyzer": "object",
      "yandexLemmerAnalyzer": "object"
      // end of the list of possible fields
    },
    "vectorSearchIndex": {
      "docEmbedderUri": "string",
      "queryEmbedderUri": "string",
      "chunkingStrategy": {
        // Includes only one of the fields `staticStrategy`
        "staticStrategy": {
          "maxChunkSizeTokens": "string",
          "chunkOverlapTokens": "string"
        }
        // end of the list of possible fields
      }
    },
    "chunkingStrategy": {
      // Includes only one of the fields `staticStrategy`
      "staticStrategy": {
        "maxChunkSizeTokens": "string",
        "chunkOverlapTokens": "string"
      }
      // end of the list of possible fields
    },
    "normalizationStrategy": "string",
    "combinationStrategy": {
      // Includes only one of the fields `meanCombination`, `rrfCombination`
      "meanCombination": {
        "meanEvaluationTechnique": "string",
        "weights": [
          "string"
        ]
      },
      "rrfCombination": {
        "k": "string"
      }
      // end of the list of possible fields
    }
  }
  // end of the list of possible fields
}
```

Represents a search index used to store and query data, either using traditional keyword-based text search or vector-based search mechanisms.

#|
||Field | Description ||
|| id | **string**

Unique identifier of the search index. ||
|| folderId | **string**

ID of the folder that the search index belongs to. ||
|| name | **string**

Name of the search index. ||
|| description | **string**

Description of the search index. ||
|| createdBy | **string**

Identifier of the subject who created this search index. ||
|| createdAt | **string** (date-time)

Timestamp representing when the search index was created.

String in [RFC3339](https://www.ietf.org/rfc/rfc3339.txt) text format. The range of possible values is from
`0001-01-01T00:00:00Z` to `9999-12-31T23:59:59.999999999Z`, i.e. from 0 to 9 digits for fractions of a second.

To work with values in this field, use the APIs described in the
[Protocol Buffers reference](https://developers.google.com/protocol-buffers/docs/reference/overview).
In some languages, built-in datetime utilities do not support nanosecond precision (9 digits). ||
|| updatedBy | **string**

Identifier of the subject who last updated this search index. ||
|| updatedAt | **string** (date-time)

Timestamp representing the last time this search index was updated.

String in [RFC3339](https://www.ietf.org/rfc/rfc3339.txt) text format. The range of possible values is from
`0001-01-01T00:00:00Z` to `9999-12-31T23:59:59.999999999Z`, i.e. from 0 to 9 digits for fractions of a second.

To work with values in this field, use the APIs described in the
[Protocol Buffers reference](https://developers.google.com/protocol-buffers/docs/reference/overview).
In some languages, built-in datetime utilities do not support nanosecond precision (9 digits). ||
|| expirationConfig | **[ExpirationConfig](#yandex.cloud.ai.common.ExpirationConfig2)**

Configuration for the expiration of the search index, defining when and how the search index will expire. ||
|| expiresAt | **string** (date-time)

Timestamp representing when the search index will expire.

String in [RFC3339](https://www.ietf.org/rfc/rfc3339.txt) text format. The range of possible values is from
`0001-01-01T00:00:00Z` to `9999-12-31T23:59:59.999999999Z`, i.e. from 0 to 9 digits for fractions of a second.

To work with values in this field, use the APIs described in the
[Protocol Buffers reference](https://developers.google.com/protocol-buffers/docs/reference/overview).
In some languages, built-in datetime utilities do not support nanosecond precision (9 digits). ||
|| labels | **object** (map<**string**, **string**>)

Set of key-value pairs that can be used to organize and categorize the search index. ||
|| textSearchIndex | **[TextSearchIndex](#yandex.cloud.ai.assistants.v1.searchindex.TextSearchIndex)**

Keyword-based text search index configuration.
This type of index is used for traditional text search, where documents are indexed based on their keywords.

Includes only one of the fields `textSearchIndex`, `vectorSearchIndex`, `hybridSearchIndex`.

Type of the search index. It can be either a traditional keyword-based text search or a vector-based search. ||
|| vectorSearchIndex | **[VectorSearchIndex](#yandex.cloud.ai.assistants.v1.searchindex.VectorSearchIndex)**

Vector-based search index configuration.
This type is used for vector search, where documents are indexed using vector embeddings.

Includes only one of the fields `textSearchIndex`, `vectorSearchIndex`, `hybridSearchIndex`.

Type of the search index. It can be either a traditional keyword-based text search or a vector-based search. ||
|| hybridSearchIndex | **[HybridSearchIndex](#yandex.cloud.ai.assistants.v1.searchindex.HybridSearchIndex)**

Hybrid (vector-based + keyword-based) search index configuration
This type is used for hybrid search, where documents are indexed using both keyword-based and vector-based search mechanisms.

Includes only one of the fields `textSearchIndex`, `vectorSearchIndex`, `hybridSearchIndex`.

Type of the search index. It can be either a traditional keyword-based text search or a vector-based search. ||
|#

## ExpirationConfig {#yandex.cloud.ai.common.ExpirationConfig2}

#|
||Field | Description ||
|| expirationPolicy | **enum** (ExpirationPolicy)

- `EXPIRATION_POLICY_UNSPECIFIED`
- `STATIC`
- `SINCE_LAST_ACTIVE` ||
|| ttlDays | **string** (int64) ||
|#

## TextSearchIndex {#yandex.cloud.ai.assistants.v1.searchindex.TextSearchIndex}

Defines the configuration for a traditional keyword-based text search index.

#|
||Field | Description ||
|| chunkingStrategy | **[ChunkingStrategy](#yandex.cloud.ai.assistants.v1.searchindex.ChunkingStrategy)**

Chunking strategy used to split text into smaller chunks before indexing.
In the case of text search, tokens are individual text characters. ||
|| ngramTokenizer | **[NgramTokenizer](#yandex.cloud.ai.assistants.v1.searchindex.NgramTokenizer)**

Tokenizer that generates n-grams.

Includes only one of the fields `ngramTokenizer`, `standardTokenizer`.

Tokenizer type used for text search. The tokenizer determines how the
input text is broken down into tokens before indexing.
If not specified, the default tokenizer configuration is applied. ||
|| standardTokenizer | **object**

Tokenizer that generates words.

Includes only one of the fields `ngramTokenizer`, `standardTokenizer`.

Tokenizer type used for text search. The tokenizer determines how the
input text is broken down into tokens before indexing.
If not specified, the default tokenizer configuration is applied. ||
|| standardAnalyzer | **object**

Standard analyzer that performs common text processing operations to normalize text.

Includes only one of the fields `standardAnalyzer`, `yandexLemmerAnalyzer`.

Analyzer type used for text search. The analyzer determines how the
tokenized text is further processed before indexing.
If not specified, the default analyzer configuration is applied. ||
|| yandexLemmerAnalyzer | **object**

Specialized analyzer that uses Yandex's lemmatization technology,
particularly effective for Russian and other Slavic languages.

Includes only one of the fields `standardAnalyzer`, `yandexLemmerAnalyzer`.

Analyzer type used for text search. The analyzer determines how the
tokenized text is further processed before indexing.
If not specified, the default analyzer configuration is applied. ||
|#

## ChunkingStrategy {#yandex.cloud.ai.assistants.v1.searchindex.ChunkingStrategy}

Defines a general strategy for chunking text into smaller segments.
Currently, only StaticChunkingStrategy is supported.

#|
||Field | Description ||
|| staticStrategy | **[StaticChunkingStrategy](#yandex.cloud.ai.assistants.v1.searchindex.StaticChunkingStrategy)**

Includes only one of the fields `staticStrategy`. ||
|#

## StaticChunkingStrategy {#yandex.cloud.ai.assistants.v1.searchindex.StaticChunkingStrategy}

Defines a chunking strategy where chunks are created with a fixed maximum chunk size and an overlap between consecutive chunks.

#|
||Field | Description ||
|| maxChunkSizeTokens | **string** (int64)

The maximum number of tokens allowed in a single chunk.
Constraints: must be within the range [100, 2048].
Default value: 800 ||
|| chunkOverlapTokens | **string** (int64)

The number of tokens that should overlap between consecutive chunks.
This allows for some context from the previous chunk to be included in the next chunk.
Constraints: must be less than or equal to half of `max_chunk_size_tokens`.
Default value: 400 ||
|#

## NgramTokenizer {#yandex.cloud.ai.assistants.v1.searchindex.NgramTokenizer}

Configuration for the NgramTokenizer, which splits text into overlapping character sequences (n-grams) of specified lengths.

Example:
Input text: `hello`
min_gram = 2, max_gram = 3

Generated tokens:
* For n = 2 (2-character n-grams): `he`, `el`, `ll`, `lo`
* For n = 3 (3-character n-grams): `hel`, `ell`, `llo`

Final tokens: `[he, el, ll, lo, hel, ell, llo]`

#|
||Field | Description ||
|| minGram | **string** (int64)

Minimum length of characters in a gram. Defaults to 3 ||
|| maxGram | **string** (int64)

Maximum length of characters in a gram. Defaults to 4 ||
|#

## VectorSearchIndex {#yandex.cloud.ai.assistants.v1.searchindex.VectorSearchIndex}

Defines the configuration for a vector-based search index. This type uses embeddings to represent documents and queries.

#|
||Field | Description ||
|| docEmbedderUri | **string**

The [ID of the model](/docs/foundation-models/concepts/embeddings) to be used for obtaining document text embeddings. ||
|| queryEmbedderUri | **string**

The [ID of the model](/docs/foundation-models/concepts/embeddings) to be used for obtaining query text embeddings. ||
|| chunkingStrategy | **[ChunkingStrategy](#yandex.cloud.ai.assistants.v1.searchindex.ChunkingStrategy)**

Chunking strategy used to split text into smaller chunks before indexing.
In the case of vector search, tokens are produced by the tokenizer from the embedding model. ||
|#

## HybridSearchIndex {#yandex.cloud.ai.assistants.v1.searchindex.HybridSearchIndex}

Defines the configuration for a hybrid (vector-based + keyword-based) search index. This type uses both embeddings and keyword-based search to represent documents and queries.

#|
||Field | Description ||
|| textSearchIndex | **[TextSearchIndex](#yandex.cloud.ai.assistants.v1.searchindex.TextSearchIndex)**

Configuration for a traditional keyword-based text search index. ||
|| vectorSearchIndex | **[VectorSearchIndex](#yandex.cloud.ai.assistants.v1.searchindex.VectorSearchIndex)**

Configuration for a vector-based search index. ||
|| chunkingStrategy | **[ChunkingStrategy](#yandex.cloud.ai.assistants.v1.searchindex.ChunkingStrategy)**

Common chunking strategy that applies to both text and vector search indexes.
If provided, it overrides the individual chunking strategies in both `text_search_index` and `vector_search_index`.
In this case, both text and vector search will use token-based chunking, where tokens are produced by the tokenizer of the embedding model. ||
|| normalizationStrategy | **enum** (NormalizationStrategy)

Normalization strategy for relevance scores from different indices. Default is MIN_MAX_STRATEGY

- `NORMALIZATION_STRATEGY_UNSPECIFIED`
- `MIN_MAX`: https://en.wikipedia.org/wiki/Feature_scaling#Rescaling_(min-max_normalization)
- `L2`: https://en.wikipedia.org/wiki/Cosine_similarity#L2-normalized_Euclidean_distance ||
|| combinationStrategy | **[CombinationStrategy](#yandex.cloud.ai.assistants.v1.searchindex.CombinationStrategy)**

Combination strategy for merging rankings from different indices. Default is arithmetic mean ||
|#

## CombinationStrategy {#yandex.cloud.ai.assistants.v1.searchindex.CombinationStrategy}

Combination strategy for merging rankings from different indices

#|
||Field | Description ||
|| meanCombination | **[MeanCombinationStrategy](#yandex.cloud.ai.assistants.v1.searchindex.MeanCombinationStrategy)**

Includes only one of the fields `meanCombination`, `rrfCombination`. ||
|| rrfCombination | **[ReciprocalRankFusionCombinationStrategy](#yandex.cloud.ai.assistants.v1.searchindex.ReciprocalRankFusionCombinationStrategy)**

Includes only one of the fields `meanCombination`, `rrfCombination`. ||
|#

## MeanCombinationStrategy {#yandex.cloud.ai.assistants.v1.searchindex.MeanCombinationStrategy}

#|
||Field | Description ||
|| meanEvaluationTechnique | **enum** (MeanEvaluationTechnique)

Technique for averaging relevance scores from different indices. Default is ARITHMETIC

- `MEAN_EVALUATION_TECHNIQUE_UNSPECIFIED`
- `ARITHMETIC`: https://en.wikipedia.org/wiki/Arithmetic_mean
- `GEOMETRIC`: https://en.wikipedia.org/wiki/Geometric_mean
- `HARMONIC`: https://en.wikipedia.org/wiki/Harmonic_mean ||
|| weights[] | **string**

Weights used for evaluating the weighted mean of relevance scores. The sum of the values must equal 1.0
If not provided, all scores are given equal weight ||
|#

## ReciprocalRankFusionCombinationStrategy {#yandex.cloud.ai.assistants.v1.searchindex.ReciprocalRankFusionCombinationStrategy}

https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf

#|
||Field | Description ||
|| k | **string** (int64)

The parameter k for RRFscore. Default is 60 ||
|#