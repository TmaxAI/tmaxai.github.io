# Global to Local Memory Pointer Networks for Task Oriented Dialogue Systems

---

> *End-to-end task TODS is challenging since knowledge bases are usually large, dynamic hard to incorporate into  a learning framework. We propose the global-to-local memory pointer **(GLMP)** networks to address this issue.   [[Paper](https://openreview.net/pdf?id=ryxnHhRqFm)*]

- **Three** main components
    1. Global memory encoder
    2. Local memory decoder
    3. Shared external knowledge

    ![](2019-02-1210-f72d3bf5-eda1-45be-85bf-8745d62a32f8.46.12.png)

- **Global memory pointer** modifies  the external knowledge by softly filtering words that are not necessary for copying.
- Then, the local memory decoder first uses a **sketch RNN** to obtain sketch responses *without slot values* but sketch tags, which can be considered as learning a **latent dialogue management** to generate dialogue action template.
- Finally, the decoder generates **local memory pointer** to copy words from external knowledge and instantiate sketch tags

## 2.  GLMP Model

- *Dialogue history*

$$X = ( x_{1},..., x_{n})$$

- *Knowledge base (KB)  information*

$$B = ( b_{1},..., b_{n})$$

- System response

$$Y = ( y_{1},..., y_{m})$$

- Model Outline
    1. The global memory uses a context RNN to encode dialogue history.
    2. Then, the l**ast hidden state** is used to read the external knowledge and generate the **global memory pointer**  
    3. During the decoding stage, the local memory decoder first generates sketch responses by a **sketch RNN.**  
    4. Then the global memory pointer and the sketch RNN hidden state are passed to the external knowledge as a **filter** and a **query**  
    5. The local memory pointer can copy text from the external knowledge to replace the sketch RNN tags and obtain the final system response                            

                             

## 2.1  External Knowledge

- External Knowledge contains the **`global contextual representation`** that is *shared* with the **encoder** and the **decoder**
- To incorporate the external knowledge into a learning framework, **end-to-end memory networks (MN)** are used to store word-level information for

             -   structural KB (**KB knowledge**) 

        -   dialogue history (**dialogue memory**)

**`Global Contextual Representation`**

- KB memory

    Each element in *B* is represented in the **triplet form** as ***(Subject, Relation, Object)***

- Dialogue memory

    The dialogue context *X* is stored in the dialogue memory module, as a **triplet form**

![](2019-02-159-f3550334-1242-4a1a-91b2-7136ef3e6e6f.58.55.png)

**Triplet KB** : {(*Tom's house, distance, 3 miles*) , .. , (*Starbucks, address, 792 Bedoin St*)}

**Triplet Dialogue** : {(*$user, turn1, I*) , ($user, turn1, need), ($user, turn1, gas) ... )}

- For the two memory modules, **a bag-of-word** representation is used as the memory embedding
- During *inference*, we copy the **object word**

     -   For example, ***3 miles*** will be copied to **(*Tom's house, distance, 3 miles*)** is selected

     -   Here ***Object( . )*** denotes the function as getting the object word from a triplet

**`Knowledge read & write`**

- External knowledge is composed of a set of trainable ***embedding matrices***

$$  C = (C^{1}, ..., C^{K+1}) \\ where \\ C^{k} \in \mathbb{R}^{\left | V \right | \times d_{emb}} K \; is \; the \; maximum\; memory\; hop, \\  \left | V \right |\ is\; the\; vocabulary\; size\; and\; d_{emb}\; is\; the\; embedding\; dimension$$

           

- Denote memory in the external knowledge as

$$M = [B; X] = (m_{1}, ...\ ,m_{n+l}) \\where\;  m_{i}\; is\; one\; of\; the\; triplets\ $$

- To read the memory, the external knowledge needs an initial query vector *q_1*

     -  It can loop over *K* hops and computes the attention weights at each hop *k* using

    $$p^{k}_{i}= Softmax((q^{k})^{T} c^{k}_{i})\\where\\c^{k}_{i} =B(C^{k}(m_{i})) \in \mathbb{R}^{d_{emb}} is \; the\; embedding \; in \; i^{th}\; memory\; position, \\q^{k} is\; the\; query\; vector\; for\; hop\;  k,\; and\; B( \;) is \; the\; bag\;of\;word\; function$$

     -  Note that the ***attention weight***, ***p***,  is a soft memory attention that decides the 
        memory relevance w.r.t the query vector

- Then the model reads out the memory ***o*** by the weighted sum over ***c*** and update the query vector ***q***

$$o^{k} = \sum_{i}p^{k}_{i}c^{k+1}_{i}\; , \;\; \; \;\;\;\; q^{k+1}=q^{k}+o^{k}$$

## 2.2  Global Memory Pointer

![](Untitled-427ed9b9-a4c7-4e31-8f8e-a4e879508523.png)

1. **Context RNN** is used to model the *sequential dependency* and encode the *context X*

         -  *End-to-end MN cannot reflect the dependencies between memories (*)*

 2.  The **hidden states** are written into the external knowledge as shown in ***Figure 1(b)***

      - This can solve the above problem (*)

 3.  The **last encoder hidden stat**e serves as the query to read the external knowledge and 
      get two outputs — ***(a) global memory pointer*** and (***b) memory readout***  

 

**`Context RNN`**

- GRU is used to encode the dialogue history into the hidden states

$$H = (h^{e}_{1},\; ...\; , h^{e}_n)$$

      and the last hidden state ***h_n*** is used to query the **external knowledge** as the **encoded the dialogue history** 

- The hidden states ***H*** are written into the dialogue memory module in the external knowledge by summing up the original memory representation with the corresponding hidden states.

$$c^{K}_{i} = c^{K}_{i} + h^{e}_{m_{i}}\;\;\;\;\; if\;\; m_{i} \in X \;\; and \;\;\;\forall\;k\;\in\;[1,K+1]   $$

**`Global Memory Pointer`**

- Global memory pointer is a vector containing real values between 0 and 1.

$$G = (g_{1},...,\;g_{n+l})$$

       Unlike conventional attention mechanism that all the weights sum to one, each element in *G* is an independent 
       probability.

- Query the external knowledge using ***h_n*** until the last hop

      -  take an **inner product** 

      -  followed by the **Sigmoid function**

    $$g_{i} = Sigmoid((q^{K})^{T} c^{K}_{i})$$

- To further strengthen the global pointing ability, we add an ***auxiliary loss*** to train the global memory pointer as a **multi-label classification task**

$$G^{label} = (g_{1}^{l},...\;g_{n+l}^{l})\;\;\;\;\;\;\; where\;\;\;\;\;\;\;\;\;g_{i}^{l} = \begin{Bmatrix} 1 \;\; if\;\; Object(m_{i})\; \in \; Y\\ 0\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\; \;\; otherwise \end{Bmatrix}$$

         - define each element by checking whether the **object words** in the memory exits in the ***expected system 
            response Y***

![](Untitled-413b380d-f90d-4f34-af5b-d15ffde4aa62.png)

               Figure 3. The process of modelling the loss function

- Then the global memory pointer is trained using binary cross-entropy loss Loss_g between G and G^label

## **`Loss Function`**

$$Loss_{g} = -\sum_{i=1}^{n+l}\;[\;g^{l}_{i} \times log\; g_{i} \;+\; (1-g^{l}_{i})\;\times\;log\;(1-g_{i})\;] $$

- Lastly, the memory readout, **q^K+1** is used as the encoded KB information

$$Memory \;\; readout \; = \; q^{K+1}$$

## 2.3  Local Memory Decoder

![](Untitled-51536bb6-c71a-4d44-89fb-b3c28c254ba2.png)

- From the Global memory encoder, we found

![](Untitled-5820d9e6-c498-4043-bb1b-8b88f3f0f059.png)

1. Local memory decoder initializes its **sketch RNN** using the concatenation of the ***h_n*** and ***q^K+1***
2. This generates a sketch response with the ***sketch tags*** but ***without slot values***
    - Eg. sketch RNN would generate     "***@poi*** is ***@distance*** away "    instead of    "***Starbucks*** is ***1 mile*** away"

3. At each decoidng time step, the hidden state of the ***sketch RNN*** is used for ***two purposes:***

    (a) Predict the next token in vocabulary , which is same as standard Sequence-to-Sequence ***(S2S)***

    (b) Serve as a vector to query the external knowledge

     -   If a ***sketch tag*** is generated, ***G*** is passed to the ***external knowledge*** and the expected output word will be 
         picked up from the ***local memory pointer***

     -   Otherwise the output word is the word that generated by the sketch RNN ***(S2S)***

**`Sketch RNN`**

    Use **GRU** to generate a ***sketech reponse*** ***Y,*** without ***real slot values***

$$Y^{s} = (y^{s}_{1},...\;y^{s}_{m})$$

- Sketch RNN learns to generate a dialogue ***action template*** based on encoded **dialogue histroy** and **KB information**
- At each decoding time step t, the sketch RNN ***hidden state*** (h_d**)** and its **output distribution (**P_vocab) are defined as

$$h^{d}_{t}\; = \;GRU(C^{1}\;(\widehat{y}_{t-1}),\;h^{d}_{t-1}), \;\;\;\;\;P^{vocab}_{t} \;=\;Softmax(Wh^{d}_{t}) $$

- Use standard **cross-entropy loss** to train the sketch RNN

$$Loss\_{v} =\sum_{t=1}^{m} -log(P^{vocab}_{t}\;(y^{s}_{t}))$$

- We replace the slot values in ***Y*** into sketch tags based on the provided entity table.
- The ***sketch tags (ST)*** are all possible slot types that start with a special token, for exampel, @*address* stands for all the address information

**`Local Memory Pointer`**

    Local memory pointer contains a sequence of pointers 

$$L=(L_{1},....\;L_{m})$$

![](Untitled-51536bb6-c71a-4d44-89fb-b3c28c254ba2.png)

- At each time step t, the **global memory pointer *G*** modifies the ***global contextual representation*** using its attnetion weights

$$c^{k}_{i} = c^{k}_{i}\; \times \;g_{i} \;\;\;\;\;\;\;\forall \; i \in [1,\; n+l]\;\; and\;\; \forall k\; \in\;[1,\;K+1]  $$

- And then the sketch RNN hidden state ***h_d*** queires the external knowledge.
- The memory attention in the last hop is the corresponding local memory pointer ***L_t*** which is represneted as the memory distribution at time step t

![](Untitled-08753954-421a-43c1-abca-f6e2f7c461eb.png)

Figure 4. The representation of "***local memory pointer*** as the memory attention in the last ho

- To train the local memory pointer, a supervision on top of the last hop memory attention in the external knowledge is added.
- We first define the **position label** of local memory pointer L label at the decoding time step t as

$$L_{t}^{label} = \begin{Bmatrix} max(z) \;\; if\;\; \exists\;z\;\;s.t.\; y_{t}= Object(m_{z})\\ n+l+1\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\; \;\; otherwise \end{Bmatrix}$$

- The position ***n+l+1*** is a **null token** in the memory that allows us to calculate loss function even if ***y_t***
does not exist in the external knowledge.
- Then, the loss between *L* and *L_label* is defined as

$$Loss_{l} = \sum_{t=1}^{m}-log(L_{t}(L_{t}^{label}))$$

- Furthermore, a record ***R ∈ R^(n+l)*** is utilized to prevent from **copying same entities multiple times.**
- All the elements in R are initialized as 1 in the beginning. During the decoding stage, if a memory position has been pointed to, its corresponding position in R will be masked out. (i.e **R_i** is set to be zero)
- During the inference time, ***yˆt*** is defined as

$$\hat{y_{t}} = \begin{Bmatrix} argmax(P_{t}^{vocab}) \;\; if\;\; argmax(P^{vocab}_{t}\;\notin ST\\ Obeject(m_{argmax)(L\bigodot R)}\;\;\;\;\;\; \;\; otherwise \end{Bmatrix} \\\;\\\\\\where, \;\;\bigodot is \;\;the\;\;element-wise\;\;multiplication$$

- Lastly, all the parameters are jointly trained by minimizing the **weighted-sum of three losses**                              
(α, β, γ are hyper-parameters)

$$Loss = \alpha \;Loss_g + \beta\;Loss_v\;+\gamma\;Loss_l$$

## 3  Experiments

- Datasets
    - bABI dialogue (Bordes & Watson, 2017)

     -     Includes 5 simulated tasks in the restaurant domain

     -     Task 1-4 are about 

              1.  calling API calls

              2.  modifying API calls

              3.  recommeding options

              4.  providing additional information

      -    Task 5 is the **union** of Tasks 1-4

    - Two test sets for each task  —  one follows the **same distribution as the training set** and the other has **OOV entity values**

    ![](Untitled-48c0d3c4-9a54-4f76-95e4-f3b2e8f54010.png)

    Example of Task 3 from bABI dialogue dataset ([https://github.com/IBM/permuted-bAbI-dialog-tasks](https://github.com/IBM/permuted-bAbI-dialog-tasks))

    - Stanford multi-domain dialogue (SMD) (Eric et al., 2017)

            -    Human-Human and  multi-domain doalogue dataset

            -    **Three** distinct domains 

                1.  Calendar scheduling

                2.  Weather information retrieval

                3.  Point-of-interest navigation

     

![](Untitled-343401b1-e357-4a8a-b571-4a10ed4f0c49.png)

                                          Example of SMD dataset

**`Results`**

![](Untitled-b041641d-8b46-4ebf-8ebc-a255bcae69e4.png)

Per-response accuracy and completion rate (in the parentheses) on bAbI dialogues. GLMP achieves the least out-of-vocabulary performance drop. Baselines are reported from Query Reduction Network (Seo et al., 2017), End-to-end Memory Network (Bordes & Weston, 2017), Gated Memory Network (Liu & Perez, 2017), Point to Unknown Word (Gulcehre et al., 2016), and Memory-to-Sequence (Madotto et al., 2018)

![](Untitled-9208b428-1c07-4f4c-b7df-b16fe50b3b02.png)

In SMD dataset, our model achieves highest BLEU score and entity F1 score over baselines, including previous state-of-the-art result from Madotto et al. (2018).

**`Ablation Study`**

- The contributions of the **global memory pointer G** and the **memory writing of dialogue histroy H** are investigated for bABI OOv task and SMD (K=1)

![](Untitled-80866f8f-bc79-43dd-b760-683d3ee0502e.png)

Ablation study using single hop model. Note a 0.4% increase in T5 suggests that the ***use of G may impose too strong prior entity probability***

## 4  Conclusion

- This paper present  end-to-end trainable model called ***global-to-local memory pointer networks*** for task-oriented dialogues.
- The global memory encoder and the local memory decoder are designed to incorporate the **shared external knowledge** into the learning framework.
- We empirically show that the global and the local memory pointer are able to effectively produce system responses even in the ***out-of-vocabulary scenario***, and visualize how global memory pointer helps as well.
- As a result, our model achieves **state-of-the-art** results in both the simulated and the human-human dialogue datasets, and holds potential for extending to other tasks such as question answering and text summarization.
