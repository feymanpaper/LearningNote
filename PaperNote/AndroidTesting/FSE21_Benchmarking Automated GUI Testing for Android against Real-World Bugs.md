### Motivation:
Specifically, dozens of testing techniques and tools have been developed and demonstrated to be effective in detecting crash bugs and outperform their respective prior work in the number of detected crashes. However, an overarching question łHow effectively and thoroughly can these tools find crash bugs in practice?ž has not been well-explored, which requires a ground-truth benchmark with real-world bugs. Since prior studies focus on tool comparisons w.r.t. some selected apps, they cannot provide direct, in-depth answers to this question.

### Summary
该工作是和安卓UI自动化测试相关。现在已有许多自动化测试工具，但是仍然没有benchmark，很难衡量工具的有效性。因此作者构造了benchmark数据集(来自20个开源app以及支持的SOTA工具)
First, we choose open-source apps as the targets to collect critical bugs because their issue repositories are public. Specifically, we designate the importance of bugs w.r.t. their issue labels assigned by app developers themselves. We collect the bugs with critical issue labels like high-priority, blocking-release, P1-urgent. We finally construct a dataset of 52 real bugs from 20 open-source Android apps by crawling the issue repositories of 1,829 Android apps. This process took us substantial manual effort (nearly two person-months) that could not be automated. It involves manually reviewing bug reports, locating buggy code versions, building app binaries, and reproducing and validating bugs. 

### Evaluation
比较的工具: Monkey, the state-of-the-practice testing tool, and five most recent state-of-the-art ones for thorough evaluation, namely Ape , Humanoid , ComboDroid , TimeMachine and Q-testing. 

### Code & Data
https:// github.com/ the-themis-benchmarks/ home
网址中包含了App及SOTA自动化工具的运行脚本
