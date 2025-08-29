
### 📋 **Vim & Kubectl Commands Summary**

| **Context**                  | **Command**                                                                 | **Description**                                      |
|-----------------------------|------------------------------------------------------------------------------|------------------------------------------------------|
| Delete to end of line       | `d$` or `D`                                                                  | Deletes from cursor to end of line                   |
| Delete to end of word       | `dw`                                                                         | Deletes from cursor to end of current word           |
| Delete to end of paragraph  | `d}`                                                                         | Deletes from cursor to end of paragraph              |
| Visual select (characters)  | `v` → move → `d`                                                             | Select characters and delete                         |
| Visual select (lines)       | `V` → move → `d`                                                             | Select lines and delete                              |
| Visual block select         | `Ctrl + v` → move → `d`                                                     | Select column/block and delete                       |
| Kubectl wide output         | `kubectl get ds -o wide -n kube-system`                                     | Shows detailed info, may overflow terminal width     |
| Pipe to pager               | `kubectl get ds -o wide -n kube-system \| less -S`                           | View output with horizontal scroll                   |
| Enable line wrapping        | `kubectl get ds -o wide -n kube-system \| less`                              | View output with wrapped lines                       |
| Custom columns + formatting | `kubectl get ds -n kube-system -o custom-columns=... \| column -t`          | Format output into aligned columns                   |
| Export to file              | `kubectl get ds -o wide -n kube-system > output.txt`                         | Save output to file                                  |
| View in Neovim              | `nvim output.txt`                                                            | Open saved output in Neovim                          |
| Live watch with formatting  | `watch -n 2 'kubectl get ds -o wide -n kube-system \| column -t'`           | Live updates with formatted output                   |
