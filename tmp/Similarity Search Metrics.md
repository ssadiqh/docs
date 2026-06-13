# Similarity Search Metrics

## 1\. L2 Distance (Euclidean Distance)

### Definition

The L2 distance between two vectors `a` and `b` is the square root of the sum of the squared differences between corresponding elements:

L2(a,b) \= √(∑(aᵢ \- bᵢ)²)

### Properties

- The L2 distance measures the straight-line distance between two points in Euclidean space, following the principles of the Pythagorean theorem.  
- It is sensitive to both the magnitude and direction of the vectors, meaning that changes in either can significantly affect the calculated distance.  
- This distance metric is commonly used in applications involving spatial or geometric data, such as image analysis, computer vision, and geographic mapping.

### Use Case

- A common use case for L2 distance is determining the closest point to a given location in two-dimensional (2D) or three-dimensional (3D) space.  
- This approach is frequently used in computer vision tasks, where spatial proximity between features or objects plays a critical role.

---

## 2\. Dot Product (Inner Product) Similarity and Distance

### Definition

The dot product of two vectors is:

a · b \= ∑(aᵢ × bᵢ)

### Properties

- The dot product of two vectors can be positive, negative, or zero, depending on the angle between them.  
- Larger dot product values typically indicate a higher degree of similarity between the vectors, especially when they are pointing in similar directions.  
- If a distance-like metric is needed, the negative of the dot product can be used. In this case, larger (more negative) values correspond to greater dissimilarity or distance.  
- The dot product is sensitive to both the magnitude and direction of the vectors, meaning that changes in either will affect the result.  
- It is frequently used in machine learning models, such as in neural networks for computing activations or in matrix factorization for recommendation systems.

### Use Case

When the length of the vector (representing something such as relevance, confidence, or quantity) is meaningful. For instance, in recommendation systems, the direction of vectors typically indicates two products are about the same topic, but larger magnitudes might indicate that a product is more popular. In this case using dot product similarity makes sense, because one would want to recommend the products that are not only about the same topic, but also popular.

---

## 3\. Cosine Similarity and Distance

### Definition

Cosine similarity measures the cosine of the angle between two vectors:

cosine\_similarity(a,b) \= (a · b) / (||a|| × ||b||)

**Note:** Cosine similarity is a similarity metric—the greater the cosine similarity, the more similar the vectors are. To convert cosine similarity to cosine distance:

cosine\_distance(a,b) \= 1 \- cosine\_similarity(a,b)

The greater the cosine distance, the less similar the vectors are.

### Alternative Calculation

Cosine similarity can be calculated more efficiently if the vectors are normalized. To normalize a vector, divide it by its L2 norm:

norm(a) \= a / ||a||

### Properties

- Cosine similarity focuses on the orientation of vectors rather than their magnitude, measuring the cosine of the angle between them.  
- It is particularly well-suited for high-dimensional, sparse data, such as text embeddings or term-frequency vectors in natural language processing.  
- This metric is invariant to vector length, meaning that scaling a vector up or down does not affect the similarity score.

### Use Case

A common use case for cosine similarity is measuring document similarity in natural language processing (NLP), where it helps identify texts with similar content regardless of their length.

---

## Choosing the Right Metric

| Metric | Sensitive to Magnitude | Normalized | Best For |
| :---- | :---- | :---- | :---- |
| L2 Distance | ✅ Yes | ❌ No | Spatial data, clustering |
| Cosine Distance | ❌ No | ✅ Yes | Text, embeddings, NLP |
| Dot Product | ✅ Yes | ❌ No | Neural networks, recommender systems |

---

## Practical Considerations

### Normalization

Normalize vectors if you expect to only need to calculate cosine similarity. For many natural language processing tasks and text embedding models, this is the default option.

### High-Dimensional Data

- L2 distance can suffer from the 'curse of dimensionality.' Consider using a different metric or using a dimensionality reduction technique if the data exhibits high dimensionality.  
- Cosine distance often performs better in high dimensions, and is the default choice in many natural language processing and text-based tasks.  
- Dot product can be computed efficiently using matrix operations, and can be used to calculate cosine similarity if the vectors are normalized.

---

## Conclusion

Choosing the right distance metric is crucial for effective similarity search. L2 distance works well for continuous, lower-dimensional data where magnitude matters. Cosine distance excels with high-dimensional, sparse data where direction is more important than magnitude. Dot product offers computational efficiency and is useful when both magnitude and direction contribute to similarity. Understanding these trade-offs helps in selecting the most appropriate metric for your specific use case.  
