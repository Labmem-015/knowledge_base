[[Start Page|На главную]]

---
# Иерархия классов

Здесь смешано наследование и композиция, так что почини это
```mermaid
flowchart TD
	b0["BaseWindow"] --> c["Focus"]
	
	b0 --> w0
	w0["Widget"] --> f2["Frame"]
	w0 --> w1["SimpleCircle"]
	w0 --> w2["PlainText"]
	
	f2 --> w1
	f2 --> w2
```