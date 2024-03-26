---
title: Improving Level Design Through Game User Research 论文笔记
date: 2019-04-10 18:31:23
tags:
---



**Abstract**

- User interviews, game metrics, psychophysiology

- Based on the results we conclude that user interviews provide the clearest indications for improvement among the considered methodologies while metrics and biometrics add different types of information that cannot be obtained otherwise.



![1554892503361](https://i.postimg.cc/pdtGds6T/1554892503361.png)



**Three Methodologies**

1. Participant interviews with player observation by researchers. Players were interviewed for about ten minutes, using a standardized script. They also filled out a 50 item questionnaire.
2. Data collection through metrics; the game was modified to log data about user behavior and user-game interaction. We logged a large number of events such as all types of movements, attacks (including attacker and target), collection of bonus items, upgrades, downgrades and game deaths, and each single key press made by the participant.
3. Data collection through biometrics; this data was gathered from the play tester by using sensors to monitor heart rate, skin conductivity and the activity of the two facial muscles, the zygomaticus major and the corrugator supercillii.



![1554892576197](https://i.postimg.cc/tRmB6qM8/1554892576197.png)

![1554892585378](https://i.postimg.cc/L5hvWwny/1554892585378.png)



**Phase 1: Preparation of the Benchmark Levels**

This phase was intended to set a benchmark, to define a point at which professional designers would release their work for internal quality assurance.



**Phase 2: GUR Data Collection**

- **General level ratings:** each player had to rate the level they had played in terms of fun, length and difficulty using a 5-point Likert scale.

- **Interview data:** Open-ended, semi-structured interviews with players were held at the end of each level.

- **Game metric data:** This allowed for periodic tracking of the player position as well as relevant game events, such as defeating enemies, jumps, collecting bonus items, etc.

- **Biometric data:** All participants were monitored with several biometric sensors during the test sessions.

  

**Phase 3: Data Evaluation and Visualization**

**Interview GUR data:**remove irrelevant information, classified as actionable changes and non-essential changes.

![1554892840377](https://i.postimg.cc/W469q0B7/1554892840377.png)

**Metric data:** Apart from acquiring play statistics for each participant, the logs were used to create heatmaps.

![1554892849326](https://i.postimg.cc/6qSzxcZB/1554892849326.png)



**Biometric data:** For the evaluation of biometric data, each level was divided into 12 equally long sections.

![1554892919940](https://i.postimg.cc/G2LXFHrn/1554892919940.png)



![1554892947632](https://i.postimg.cc/tTB22pCZ/1554892947632.png)



**Phase 4: Modifications**

Each level was modified based on the data collected in phase 3 according to the three different pair-wise combinations of methodologies. These were:

1. interviews and game metrics; 
2. interviews and biometrics; 
3. game metrics and biometrics.

For each combination of methodologies, a total of six changes were implemented across the three levels.

![1554893080181](https://i.postimg.cc/wTy0QKKM/1554893080181.png)



**Phase 5: Evaluation of Modifications**

- Players were asked to complete the Game Experience Questionnaire (GEQ), a tool that is commonly used in the GUR field to quickly analyze player experience.
- A total of 40 participants (22 of which were female) in ages ranging from 15 to 27 years (median age of 23 years)took part in this second data collection session. Players played one version of the modified level-sets, consisting of three levels.



**Results**


![1554893180704](https://i.postimg.cc/WbBS99jn/1554893180704.png)

- On the whole, the differences between the combined methodologies were much smaller than we expected.
- A consistent pattern can be seen across the variables Positive Affect, Flow, Positive Experience, and Competence: The levels modified by `interview & metrics' were rated most positively.
- three dimensions had statistically significant differences between the groups: Positive Experience (p=.008, F=5.595), Competence (p=.04, F=3.530), and Positive Affect (p=.048, F=3.290).
- the significant differences within those dimensions are found between the pairings "interview & metrics" and  "biometrics & metrics" for the dimensions Competence (p=.03) and Positive Affect (p=.038), and between the pairings "interview & metrics" and "interview & biometrics" for the dimension Positive Experience (p=.008).
- there could be a Type II error as the ANOVA analysis was significant for only a few GEQ dimensions.
- From the point of view of a level designer, each pair of GUR methodologies was able to provide actionable indications regarding locations or situations that should be modified to improve player satisfaction.



**Deficiency**

- 第二次组织试玩没有测试metrics和biometrics数据验证。
- 关卡的修改比较主观，没有说明与数据的对应关系。
- 样板数量过小



