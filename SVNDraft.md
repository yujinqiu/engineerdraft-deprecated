##经常搞混问题
- svn update 和 svn revert
如果working copy 的版本和server 版本一样, 然后修改working copy 的内容后来发现需要废弃当前修改,此时应该使用
`
svn revert
`

如果你的working copy 很久没有修改, 然后有人修改server 端, 并且ci 进去, 导致server 端的revision >  working copy , 此时采用,直接更新revision.
`
svn update
`
