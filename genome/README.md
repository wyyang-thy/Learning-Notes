## 20260917以yang-asm1比对T2T判断breakpoint，并用petmar来做断点仲裁，以此例来深入了解基因组知识
### 基因组中的序列字符串本身永远是 5'→3' 书写的，不确定的是：这一串到底代表染色体的哪条链。所以当yang-asm1中的某一条contig作为query来minimap比对到T2T上时，以query的真实存储链向为+，如果能正好比对到reference上记录链向为+，如果需要query反向互补之后与reference才match则记录链向为-

