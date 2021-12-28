# Knuth-Morris-Pratt Algorithm

### Usage
- a string-searching algorithm
- searches for occurences of a "word" `W` within a "text string" `S`
- ususally used in string pattern search

### Pseudocode
```
let j = 0(position of current character in S)
let k = 0(position of current character in W)
let nP = 0

while j < length(S) do:
  if W[k] = S[j] then:
    let j = j + 1
    let k = k + 1
    if k = length(W) then:
      (occurence found, if only first occurence is needed, m = j - k may be return here)
      let P[nP] = j - k, nP = npP + 1
      let k = T[k]
    else:
      let k = T[k]
      if k < 0 then:
        let j = j + 1
        let k = k + 1
```
