# Milestone 1: Community and Label Taxonomy

## Community

I chose r/nba, a Reddit community focused on NBA discussion. This community is active, text-heavy, and has varied discourse quality. Some comments are detailed basketball analysis, some are bold unsupported opinions, and some are emotional reactions to games, players, trades, or highlights.

This community is a good fit because NBA fans often debate whether a comment is real analysis, a hot take, or just a reaction.

## Labels

I will use three labels:

### analysis

A post is `analysis` if it makes a structured basketball argument supported by statistics, tactical reasoning, historical comparison, or clear cause-and-effect explanation.

Examples:

1. “The Wolves’ defense worked because they kept sending early help from the weak side and forced Denver’s role players to make quick decisions.”
2. “Brunson’s scoring jump is not just higher usage. His footwork in the midrange lets him create clean looks, and his turnover rate has stayed low.”

### hot_take

A post is `hot_take` if it makes a bold or confident basketball claim without enough supporting evidence.

Examples:

1. “Tatum is never winning as the best player on a title team.”
2. “The Lakers should trade everyone except LeBron and AD. This roster is cooked.”

### reaction

A post is `reaction` if it mainly expresses emotion, surprise, frustration, hype, sarcasm, or disappointment without building an argument.

Examples:

1. “I can’t believe they blew that lead again. This team is impossible to watch.”
2. “That dunk was disgusting. He ended that man’s career.”

## Hard Edge Case

The hardest edge case is a post that makes a bold claim but includes one statistic.

Example:

“LeBron is overrated because his playoff record against top-seeded teams is below .500.”

This could look like `analysis` because it includes a stat, but it could also be `hot_take` because the stat is used mainly to support a provocative claim.

## Decision Rule

If a post includes evidence but does not explain why the evidence supports the claim, I will label it `hot_take`.

If the post connects the evidence to a broader basketball argument, explains context, or gives clear reasoning, I will label it `analysis`.

If the post mainly expresses emotion without making a real argument, I will label it `reaction`.