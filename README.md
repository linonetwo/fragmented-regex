# fragmented-regex

Maintainable regex, Shareable regex, Extended regex for data extraction. Powered by tagged template literal and parser combinator.

## Usage

✍️Not implemented yet!!

```javascript
import re from 'fragmented-regex';

const stockParser = re`
  (?<method>${stockTypes})[共合]?计?${stockAmountParser}|
  ${stockAmountParser}(?<method>${stockTypes})
`;


const getResult = re`
  (${priceParser}.*${stockParser})+|
  (${stockParser}.*${priceParser})+|
  ${stockParser}
`;
```

### Domain specific parser

You may have a parser that can detect name in a string. Now you can combine it with regex like this:

```javascript
import re, { wrap } from 'fragmented-regex';
import isChineseName from 'is-chinese-name';

const chineseName = wrap(isChineseName);

const dismissParser = re`
(?P<name>${chineseName})(?P<gender>男|女).*(?P<reason>任职期将满|任期届满)
`;

```

### Thesaurus

Sometimes you may fell that you have wrote the regex you are coding before. Or you may copy && paste them from your legacy code. Never. Now you can import them from the npm.

```javascript
import re from 'fragmented-regex';


```

Local thesaurus are simple disposable dict in current context. They are not meant to be shareable.

```javascript
import re from 'fragmented-regex';

const context = {
  '。': '，。；：！？',
  元: /[欧美日]?元/,
  根据: '根据 对照 按照',
  公司: '公司 发行人',
  所从事的: /所(处|属|从事)[的]?/,
  是: '于 为 是 属 属于',
  '<>': ''
};

const 主营业务均 = re({ context })`
<所从事的>.*?行业(应归类)?|主营业务均|从事的.*业务(所属行业)?|[归隶]
`;

// (根据|对照|按照).*上市公司行业分类指引.*(公司|发行人).*(所(处|属|从事)[的]?.*?行业(应归类)?|主营业务均|从事的.*业务(所属行业)?|[归隶])?(于|为|是|属(于)?).*?(第)?(“)?(?P<csrc_industry_code>[A-Z][\d]{2})(?P<csrc_industry_name>.*)(的|”||，|。)
const moneyParser = re({ context })`
<根据>.*上市公司行业分类指引.*<公司>.*${主营业务均}?<是>.*?(第)?(“)?(?P<csrc_industry_code>[A-Z][\d]{2})(?P<csrc_industry_name>.*)(的|”||，|。)
`;

```

Parser is now two dimensional, one is the pattern, another is the thesaurus.

### Debugging

Sometimes regex stops working and it's too complex to figure out.

We can reduce parser to smaller fragments so that we can use Exclusion: "These fragments all matched, so the only fragment that does not match, is what we should investigate in."

To split parser into smaller fragments, we imply:

1. We regard strings as anchor and anchor is joined by .*
1. In a group, we join different mutation of a pattern by |

There are other benefits:

1. We can get statistic data about each fragments' matching result
1. Fragments' matching result can be stored into a dictionary for later AI training

### Obstacles (RFC)

After a pattern was wrote, it will be blocked by some slight mutation in natural language from time to time.

You may write ```根据(?P<underwriting_policy>[^，]*规定)，``` at the first time.

Then change it to ```根据(?P<underwriting_policy>[^，]*?(的)?规定)，``` due to some mutation in natural language.

And end up with ```根据(?P<underwriting_policy>[^，]*?(的)?(规定)?)，```.

Those ```(的)?(规定)?``` are obstacles that blocks parsing.

#### Partition based tolerance

If there is a literal based fragment (especially chinese literals), and partition based Levenshtein distance below a threshold, then this fragment is regarded matched.

## Motivation

Dealing with string was a wordy job. For example, CSS and RegExp, they were derived from business needs, suffered from rapid change and will eventually grow into an unrefactorable enormous string.

Now there are [styled-components](https://github.com/styled-components/styled-components) for CSS, which allow [styled-flex-component](https://github.com/SaraVieira/styled-flex-component) and [rebass](https://github.com/jxnblk/rebass) to emerge. So CSS can be npm-publishable strings that are business-independent.

And I really need a similar thing for RegExp.

It should:

- Allow refactor of literals and regex groups
- Handle obstacle that blocks parsing
- Generate example for visual debugging and snapshot testing (for regression)
- Tells why it doesn't match
- Parser looks like an example that it will match

## Design Philosophy

- Use business-independent block to build pattern that reads similar to some string appears in your business
- Utilize human's builtin pattern matching when debugging
- Can include optional metadata to enable advanced usages

Got inspired in [this thread](https://github.com/GregRos/parjs/issues/4#issuecomment-379176500).
