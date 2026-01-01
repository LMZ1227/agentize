- When writing any scripts, **AVOID** being shell-specific.
  - The most common thing I can foresee is that `BASH_SOURCE[0]` or `$0` between zsh and bash to get the current script name.
    Use `AGENTIZE_HOME` as a absolute path to refer to the root of this project instead.
