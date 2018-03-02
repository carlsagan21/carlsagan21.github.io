---
title: Automated program corrector for programming assignments using Deep Learning
---

## abstract

In this article, I will explain how to use deep learning to find and correct wrong programs when there are many correct answers. First, I will introduce the idea of how to proceed with learning, the learning model used, and show that the results of the learning can be used to detect and correct errors in programs. Then, I will point out the theoretical limitations of the current method and end the presentation by suggesting improvement directions derived from the limitations pointed out.

## Introduction

Before starting the presentation, I first make clear that the ideas presented here are in the published paper[^pu2016sk_p] and that I did not originally devise them. Rather, I have focused on understanding the concept of deep learning and natural language processing which were both unfamiliar to me, and reproducing the result of the paper using libraries. In addition, I focused on clarifying the significance of the paper by digesting the theoretical background of it, which was not mentioned by the authors of the original paper intentionally or not.

### train set generation

The most important idea and premise of the learning used in this paper is that the program code can be manipulated using natural language processing techniques. Since program code and natural languages are similar[^hindle2012naturalness], applying matured techniques from the alien field is worth a try. If it is possible to think of a program as a natural language, it is possible to generate the code using techniques for generating languages in NLP techniques. More specifically, using a model for translation in the NLP makes it possible to generate original codes from a given code. Learning program using the translation model takes advantage of NLP techniques in two ways. The program is regarded as a sequence divided into tokens, and an n-gram idea is used to input the generation condition.

The goal of learning is to ensure that the correct code is generated when you insert the creation conditions. To do this, put 'before and after program lines to be created' as the creation conditions, and make sure 'a line of correct program' as the result. Here's how to implement a specific implementation. First, converts the code into a sequence of tokens separated by lines, and inserts lines to indicate the start and the end of the program. Then set the three lines of code as one set, lines 1 and 3 as input, and line 2 as output. Perform this procedure for all programs. Then set the two lines of code as one set, two lines as input, and the empty result (Sigma) as output. If you repeat this process, the whole learning set is constructed. As a result, if you have a 10-line program, it becomes 12 lines by the starting and ending lines, from which 10 and 11 sets are derived, resulting in a total of 21 learning sets. Up to this point, we have tokenized a program flat to the sequence level and used a modified 2-gram at the line level by referencing the line before and after.

### building model and training

And I constructed and trained the deep learning model, specifically modifying the seq2seq translation model to support 2-gram line inputs. As a result, I was able to see that training be successful and that the correct results were being generated.

### code generation and error localization

Now that I have successfully generated the code, I need to find the error in the original program. If a line is the cause of the error, you can fix the problem by regenerating or removing the line. Error localization can be done stochastically from generated code. I made the probability of the generated code for each generation point is derived also. And tried to modify the code in descending order according to probability. Assuming that there are 21 possibilities in 10-line code, there are 21 points for one point correction and $ \comb{21}{2} $ for 2 point correction and so on. Thus, the size of the search space is theoretically $ 2^{21} $. In addition, I can use beam search to generate n possible lines stochastically instead of generating one for each revision point, which greatly increases the search space but predictably reduces the chance of missing the answer.

## issues

However, the above method cannot fundamentally go beyond the theoretical limits. First, the logic to find the location of the error is incomplete - it just relies on stochastic results of code generation, which is somewhat irrelevant with occasions of errors; second, it does not exploit the difference between the program and the natural language, thus throwing out all the rich information of programs.

Though the method might succeed in regenerating codes, it cannot locate exactly in which line a program is wrong and whether generate a fix. The suggested localizing method is brute-forcing, which is quite unpleasant considering the search area growing exponentially. This can be dramatically improved coordination with AA team, MaxSAT based error localization system.

Next, as adapting methodologies of NLP, I threw out all the rich information of program codes compared to natural languages. Alpha conversion, AST[^mou2016convolutional], type and further information using static analysis are the properties which only belongs to programs. These properties can be potential candidates to increase the performance of the corrector.

In my opinion, exploiting AST structure is the best next step to take.

---

[PPT](https://github.com/carlsagan21/seminar-template/blob/master/Automated%20program%20corrector/sookim_170908.pptx): most up to date explaination
[Latex](https://github.com/carlsagan21/seminar-template/tree/master/Automated%20program%20corrector): originally written in latex
[Code](https://github.com/carlsagan21/tensorflow-practice/blob/master/corrector.py): can be changed

[^hindle2012naturalness]: A. Hindle, E. T. Barr, Z. Su, M. Gabel, and P. Devanbu. On the naturalness of software. In *Software Engineering (ICSE), 2012 34th International Conference on*, pages 837-847. IEEE. 2012.

[^mou2016convolutional]: A. Hindle, E. T. Barr, Z. Su, M. Gabel, and P. Devanbu. On the naturalness of software. In Software Engineering (ICSE), 2012 34thL. Mou, G. Li, L. Zhang, T. Wang, and Z. Jin. Convolutional neural networks over tree structures for programming language processing. In *AAAI*, pages 1287-1293, 2016.

[^pu2016sk_p]: A. Hindle, E. T. Barr, Z. Su, M. Gabel, and P. Devanbu. On the naturalness of software. In Software Engineering (ICSE), 2012 34thY. Pu, K. Narasimhan, A. Solar-Lezama, and R. Barzilay. sk p: a neural program corrector for moocs. In *Companion Proceedings of the 2016 ACM SIGPLAN International Conference on Systems, Programming, Languages and Applications: Software for Humanity*, pages 39-40. ACM, 2016.
