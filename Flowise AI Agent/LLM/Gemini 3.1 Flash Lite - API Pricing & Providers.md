---
title: "Gemini 3.1 Flash Lite - API Pricing & Providers"
source: "https://openrouter.ai/google/gemini-3.1-flash-lite"
author:
created: 2026-06-27
description: "Gemini 3.1 Flash Lite is Google’s GA high-efficiency multimodal model optimized for low-latency, high-volume workloads. $0.25 per million input tokens, $1.50 per million output tokens. 1,048,576 token context window, maximum output of 65,536 tokens. Higher uptime with 2 providers."
---
## Google: Gemini 3.1 Flash Lite

[Compare](https://openrouter.ai/compare/google/gemini-3.1-flash-lite)

Gemini 3.1 Flash Lite is Google’s GA high-efficiency multimodal model optimized for low-latency, high-volume workloads. It supports text, image, video, audio, and PDF inputs, and is designed for lightweight agentic workflows, simple data extraction, and applications where responsiveness and API cost are the primary constraints.

Supports full thinking levels (minimal, low, medium, high) for fine-grained cost/performance trade-offs. Priced at half the cost of Gemini 3 Flash.

Modalities

In / Out Price

$0.25 / $1.50per 1M

Context

1M

Released

May 7, 2026

[Compare](https://openrouter.ai/compare/google/gemini-3.1-flash-lite)

## Providers

Different companies host the same model. OpenRouter routes your request to one of them based on the routing mode you pick — Balanced (price + speed), Nitro (fastest), or Exacto (one fixed provider).

| Provider | Input /M | Output /M | Cache Read /M | Audio Cache /M | Latency | Throughput | Uptime |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  | $0.25 | $1.50 | $0.025 | $0.05 | 0.82s | 86 tps | 99.06% |
|  | $0.25 | $1.50 | $0.025 | $0.05 | 0.61s | 92 tps | 99.41% |

## Effective Pricing

The chart below shows the average price customers are actually paying after prompt caching. Depending on the amount of repeated context you send, this can be 60–80% cheaper than the provider list price. Shown are rolling averages from the past 30 days.

### Weighted Average

Weighted Avg Input Price

$0.186

/M tokens

Weighted Avg Output Price

$1.50

/M tokens

| Provider | Input $/1M | Output $/1M | Cache hit rate | Token share |
| --- | --- | --- | --- | --- |
| Google Vertex | $0.215 | $1.50 | 17.9% | 56.7% |
| Google AI Studio | $0.150 | $1.48 | 45.1% | 43.3% |

### Input Price / 1M tokens (7 days)

<svg role="application" tabindex="0" width="483" height="320" viewBox="0 0 483 320" style="width: 100%; height: 100%; display: block;"><defs><clipPath id="recharts10378-clip"><rect x="32" y="24" height="242" width="451"></rect></clipPath></defs><g><g><g><text height="30" orientation="bottom" width="451" stroke="none" x="56" y="280" text-anchor="middle" fill="#666"><tspan x="56" dy="0.71em">Jun 20</tspan></text></g> <g><text height="30" orientation="bottom" width="451" stroke="none" x="171.14285714285714" y="280" text-anchor="middle" fill="#666"><tspan x="171.14285714285714" dy="0.71em">Jun 22</tspan></text></g> <g><text height="30" orientation="bottom" width="451" stroke="none" x="286.2857142857143" y="280" text-anchor="middle" fill="#666"><tspan x="286.2857142857143" dy="0.71em">Jun 24</tspan></text></g> <g><text height="30" orientation="bottom" width="451" stroke="none" x="401.42857142857144" y="280" text-anchor="middle" fill="#666"><tspan x="401.42857142857144" dy="0.71em">Jun 26</tspan></text></g></g></g> <g><g><g><text width="32" orientation="left" height="242" stroke="none" x="24" y="266" text-anchor="end" fill="#666"><tspan x="24" dy="0.355em">0</tspan></text></g> <g><text width="32" orientation="left" height="242" stroke="none" x="24" y="205.5" text-anchor="end" fill="#666"><tspan x="24" dy="0.355em">0.1</tspan></text></g> <g><text width="32" orientation="left" height="242" stroke="none" x="24" y="145" text-anchor="end" fill="#666"><tspan x="24" dy="0.355em">0.1</tspan></text></g> <g><text width="32" orientation="left" height="242" stroke="none" x="24" y="84.5" text-anchor="end" fill="#666"><tspan x="24" dy="0.355em">0.2</tspan></text></g> <g><text width="32" orientation="left" height="242" stroke="none" x="24" y="24" text-anchor="end" fill="#666"><tspan x="24" dy="0.355em">0.2</tspan></text></g></g> <text offset="5" x="16" y="145" text-anchor="middle" fill="#808080" style="text-anchor: middle;"><tspan x="16" dy="0.355em">$/1M</tspan></text></g><g><defs><clipPath id="clipPath-recharts-line-_r_1cq_"><rect x="-193.5" y="24" width="902" height="242"></rect></clipPath></defs><path name="Google AI Studio" stroke="#0088FE" stroke-dasharray="407.8910217285156px 0px" stroke-width="2" fill="none" id="recharts-line-_r_1cq_" height="242" width="451" clip-path="url(#clipPath-recharts-line-_r_1cq_)" d="M56,107.744C75.19,114.523,94.381,121.302,113.571,121.302C132.762,121.302,151.952,109.041,171.143,109.041C190.333,109.041,209.524,112.121,228.714,112.121C247.905,112.121,267.095,106.667,286.286,106.667C305.476,106.667,324.667,107.159,343.857,107.159C363.048,107.159,382.238,99.486,401.429,97.258C420.619,95.029,439.81,94.408,459,93.787"></path></g><g><defs><clipPath id="clipPath-recharts-line-_r_1cr_"><rect x="-193.5" y="24" width="902" height="242"></rect></clipPath></defs><path name="Google Vertex" stroke="#00C49F" stroke-dasharray="404.91357421875px 0px" stroke-width="2" fill="none" id="recharts-line-_r_1cr_" height="242" width="451" clip-path="url(#clipPath-recharts-line-_r_1cr_)" d="M56,47.94C75.19,51.022,94.381,54.103,113.571,54.103C132.762,54.103,151.952,46.723,171.143,46.723C190.333,46.723,209.524,47.246,228.714,48.293C247.905,49.34,267.095,55.131,286.286,55.131C305.476,55.131,324.667,52.961,343.857,51.373C363.048,49.786,382.238,46.779,401.429,45.604C420.619,44.43,439.81,44.137,459,43.843"></path></g></svg>

### Output Price / 1M tokens (7 days)

<svg role="application" tabindex="0" width="483" height="320" viewBox="0 0 483 320" style="width: 100%; height: 100%; display: block;"><defs><clipPath id="recharts10379-clip"><rect x="32" y="24" height="242" width="451"></rect></clipPath></defs><g><g><g><text height="30" orientation="bottom" width="451" stroke="none" x="56" y="280" text-anchor="middle" fill="#666"><tspan x="56" dy="0.71em">Jun 20</tspan></text></g> <g><text height="30" orientation="bottom" width="451" stroke="none" x="171.14285714285714" y="280" text-anchor="middle" fill="#666"><tspan x="171.14285714285714" dy="0.71em">Jun 22</tspan></text></g> <g><text height="30" orientation="bottom" width="451" stroke="none" x="286.2857142857143" y="280" text-anchor="middle" fill="#666"><tspan x="286.2857142857143" dy="0.71em">Jun 24</tspan></text></g> <g><text height="30" orientation="bottom" width="451" stroke="none" x="401.42857142857144" y="280" text-anchor="middle" fill="#666"><tspan x="401.42857142857144" dy="0.71em">Jun 26</tspan></text></g></g></g> <g><g><g><text width="32" orientation="left" height="242" stroke="none" x="24" y="266" text-anchor="end" fill="#666"><tspan x="24" dy="0.355em">0</tspan></text></g> <g><text width="32" orientation="left" height="242" stroke="none" x="24" y="205.5" text-anchor="end" fill="#666"><tspan x="24" dy="0.355em">0.4</tspan></text></g> <g><text width="32" orientation="left" height="242" stroke="none" x="24" y="145" text-anchor="end" fill="#666"><tspan x="24" dy="0.355em">0.8</tspan></text></g> <g><text width="32" orientation="left" height="242" stroke="none" x="24" y="84.50000000000003" text-anchor="end" fill="#666"><tspan x="24" dy="0.355em">1.2</tspan></text></g> <g><text width="32" orientation="left" height="242" stroke="none" x="24" y="24" text-anchor="end" fill="#666"><tspan x="24" dy="0.355em">1.6</tspan></text></g></g> <text offset="5" x="16" y="145" text-anchor="middle" fill="#808080" style="text-anchor: middle;"><tspan x="16" dy="0.355em">$/1M</tspan></text></g><g><defs><clipPath id="clipPath-recharts-line-_r_1cs_"><rect x="-193.5" y="24" width="902" height="242"></rect></clipPath></defs><path name="Google AI Studio" stroke="#0088FE" stroke-dasharray="433.0790100097656px 0px" stroke-width="2" fill="none" id="recharts-line-_r_1cs_" height="242" width="451" clip-path="url(#clipPath-recharts-line-_r_1cs_)" d="M56,63.188C75.19,81.671,94.381,100.155,113.571,100.155C132.762,100.155,151.952,62.82,171.143,57.021C190.333,51.222,209.524,51.029,228.714,48.322C247.905,45.615,267.095,40.78,286.286,40.78C305.476,40.78,324.667,47.79,343.857,47.79C363.048,47.79,382.238,47.109,401.429,46.087C420.619,45.065,439.81,43.362,459,41.659"></path></g><g><defs><clipPath id="clipPath-recharts-line-_r_1ct_"><rect x="-193.5" y="24" width="902" height="242"></rect></clipPath></defs><path name="Google Vertex" stroke="#00C49F" stroke-dasharray="403.2790222167969px 0px" stroke-width="2" fill="none" id="recharts-line-_r_1ct_" height="242" width="451" clip-path="url(#clipPath-recharts-line-_r_1ct_)" d="M56,40.213C75.19,41.821,94.381,43.429,113.571,43.429C132.762,43.429,151.952,39.395,171.143,39.395C190.333,39.395,209.524,39.603,228.714,39.603C247.905,39.603,267.095,39.274,286.286,39.216C305.476,39.159,324.667,39.13,343.857,39.13C363.048,39.13,382.238,39.151,401.429,39.17C420.619,39.189,439.81,39.217,459,39.245"></path></g></svg>

## Performance

Throughput is how fast the model writes (tokens per second — higher is better). Latency is total round-trip time (lower is better). TTFT is time-to-first-token — how long before you see anything appear (lower is better).

Throughput

92tok/s

best across providers

Latency

0.60s

p50, best provider

### Throughput

Google AI Studio

Avg105 tok/s

Google Vertex

Avg99 tok/s

### Latency

Google AI Studio

Avg0.61 s

Google Vertex

Avg1.06 s

### E2E Latency

Google AI Studio

Avg1.72 s

Google Vertex

Avg2.84 s

## Uptime

Percent of requests that succeeded over the last 30 days. OpenRouter monitors every provider continuously and automatically retries on the next-best provider when one returns an error.

Avg. Provider Uptime (3d)

98.59%

averaged across all endpoints

<iframe title="uptime graph" src="https://us5.datadoghq.com/graph/embed?token=9fc4419a875d135a6741fb33ec0fa989b95267ff66a0b7dc8c00cc811ec276d2&amp;legend=true&amp;height=318&amp;width=980" width="980" height="318"></iframe>

When an error occurs in an upstream provider, we can recover by routing to another healthy provider, if your request filters allow it. You can access uptime data programmatically through the [Endpoints API](https://openrouter.ai/docs/api/api-reference/endpoints/list-endpoints). [Learn more](https://openrouter.ai/docs/provider-routing) about our load balancing and customization options.

<iframe title="finish reason graph" src="https://us5.datadoghq.com/graph/embed?token=233d879680853f2752b73e045d4f5764d243d4570decb8b717506622aa376be4&amp;legend=true&amp;height=318&amp;width=980" width="980" height="318"></iframe>

## Apps

Public apps that send the most traffic to this model. Good signal for what real production workloads look like — and a hint at which use cases this model is best suited for.

## Activity

Token volume and request traffic to this model over time.

<svg role="application" tabindex="0" width="702" height="320" viewBox="0 0 702 320" style="width: 100%; height: 100%; display: block;"><defs><clipPath id="recharts6963-clip"><rect x="42" y="0" height="290" width="660"></rect></clipPath></defs><defs><pattern id="forecastStripes" patternUnits="userSpaceOnUse" width="6" height="6" patternTransform="rotate(45)"><rect width="6" height="6" fill="#A3A3A3"></rect><rect width="3" height="6" fill="white" fill-opacity="0.15"></rect></pattern></defs><g><g><g><text font-size="12" height="30" orientation="bottom" width="660" stroke="none" x="57.483870967741936" y="304" text-anchor="middle" fill="#666"><tspan x="57.483870967741936" dy="0.71em">May 28</tspan></text></g> <g><text font-size="12" height="30" orientation="bottom" width="660" stroke="none" x="141.35483870967744" y="304" text-anchor="middle" fill="#666"><tspan x="141.35483870967744" dy="0.71em">Jun 1</tspan></text></g> <g><text font-size="12" height="30" orientation="bottom" width="660" stroke="none" x="225.22580645161293" y="304" text-anchor="middle" fill="#666"><tspan x="225.22580645161293" dy="0.71em">Jun 5</tspan></text></g> <g><text font-size="12" height="30" orientation="bottom" width="660" stroke="none" x="309.0967741935484" y="304" text-anchor="middle" fill="#666"><tspan x="309.0967741935484" dy="0.71em">Jun 9</tspan></text></g> <g><text font-size="12" height="30" orientation="bottom" width="660" stroke="none" x="392.9677419354839" y="304" text-anchor="middle" fill="#666"><tspan x="392.9677419354839" dy="0.71em">Jun 13</tspan></text></g> <g><text font-size="12" height="30" orientation="bottom" width="660" stroke="none" x="476.8387096774194" y="304" text-anchor="middle" fill="#666"><tspan x="476.8387096774194" dy="0.71em">Jun 17</tspan></text></g> <g><text font-size="12" height="30" orientation="bottom" width="660" stroke="none" x="560.7096774193549" y="304" text-anchor="middle" fill="#666"><tspan x="560.7096774193549" dy="0.71em">Jun 21</tspan></text></g> <g><text font-size="12" height="30" orientation="bottom" width="660" stroke="none" x="644.5806451612904" y="304" text-anchor="middle" fill="#666"><tspan x="644.5806451612904" dy="0.71em">Jun 25</tspan></text></g></g></g> <g><g><g></g><g><text font-size="12" orientation="left" width="60" height="290" stroke="none" x="34" y="217.5" text-anchor="end" fill="#666"><tspan x="34" dy="0.355em">20B</tspan></text></g> <g><text font-size="12" orientation="left" width="60" height="290" stroke="none" x="34" y="145" text-anchor="end" fill="#666"><tspan x="34" dy="0.355em">40B</tspan></text></g> <g><text font-size="12" orientation="left" width="60" height="290" stroke="none" x="34" y="72.5" text-anchor="end" fill="#666"><tspan x="34" dy="0.355em">60B</tspan></text></g> <g><text font-size="12" orientation="left" width="60" height="290" stroke="none" x="34" y="9" text-anchor="end" fill="#666"><tspan x="34" dy="0.355em">80B</tspan></text></g></g></g><g id="recharts-bar-_r_1ag_"><g><g><g><path fill="#0088FE" name="undefined" x="48" y="100.148982499125" width="18.870967741935484" height="189.851017500875" radius="0" d="M 48,100.148982499125 h 18.870967741935484 v 189.851017500875 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="68.96774193548387" y="94.08250993087499" width="18.870967741935484" height="195.917490069125" radius="0" d="M 68.96774193548387,94.08250993087499 h 18.870967741935484 v 195.917490069125 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="89.93548387096774" y="166.77444059462502" width="18.870967741935484" height="123.22555940537498" radius="0" d="M 89.93548387096774,166.77444059462502 h 18.870967741935484 v 123.22555940537498 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="110.90322580645162" y="169.70806122624998" width="18.870967741935484" height="120.29193877375002" radius="0" d="M 110.90322580645162,169.70806122624998 h 18.870967741935484 v 120.29193877375002 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="131.8709677419355" y="109.70569718949999" width="18.870967741935484" height="180.29430281050003" radius="0" d="M 131.8709677419355,109.70569718949999 h 18.870967741935484 v 180.29430281050003 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="152.83870967741936" y="85.31836387387501" width="18.870967741935484" height="204.681636126125" radius="0" d="M 152.83870967741936,85.31836387387501 h 18.870967741935484 v 204.681636126125 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="173.80645161290323" y="72.24615709849999" width="18.870967741935484" height="217.7538429015" radius="0" d="M 173.80645161290323,72.24615709849999 h 18.870967741935484 v 217.7538429015 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="194.7741935483871" y="82.55898778837499" width="18.870967741935484" height="207.441012211625" radius="0" d="M 194.7741935483871,82.55898778837499 h 18.870967741935484 v 207.441012211625 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="215.74193548387098" y="82.31374702275" width="18.870967741935484" height="207.68625297725" radius="0" d="M 215.74193548387098,82.31374702275 h 18.870967741935484 v 207.68625297725 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="236.70967741935485" y="162.37689660625" width="18.870967741935484" height="127.62310339375" radius="0" d="M 236.70967741935485,162.37689660625 h 18.870967741935484 v 127.62310339375 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="257.6774193548387" y="174.15778752825003" width="18.870967741935484" height="115.84221247174997" radius="0" d="M 257.6774193548387,174.15778752825003 h 18.870967741935484 v 115.84221247174997 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="278.64516129032256" y="92.840335806375" width="18.870967741935484" height="197.15966419362502" radius="0" d="M 278.64516129032256,92.840335806375 h 18.870967741935484 v 197.15966419362502 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="299.61290322580646" y="71.29613055812499" width="18.870967741935484" height="218.703869441875" radius="0" d="M 299.61290322580646,71.29613055812499 h 18.870967741935484 v 218.703869441875 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="320.58064516129036" y="76.56045058025" width="18.870967741935484" height="213.43954941975" radius="0" d="M 320.58064516129036,76.56045058025 h 18.870967741935484 v 213.43954941975 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="341.5483870967742" y="79.15969037674999" width="18.870967741935484" height="210.84030962325" radius="0" d="M 341.5483870967742,79.15969037674999 h 18.870967741935484 v 210.84030962325 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="362.51612903225805" y="72.069233140875" width="18.870967741935484" height="217.93076685912501" radius="0" d="M 362.51612903225805,72.069233140875 h 18.870967741935484 v 217.93076685912501 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="383.48387096774195" y="123.25022161075" width="18.870967741935484" height="166.74977838925" radius="0" d="M 383.48387096774195,123.25022161075 h 18.870967741935484 v 166.74977838925 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="404.45161290322585" y="115.69398242637502" width="18.870967741935484" height="174.30601757362498" radius="0" d="M 404.45161290322585,115.69398242637502 h 18.870967741935484 v 174.30601757362498 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="425.4193548387097" y="57.819902336750005" width="18.870967741935484" height="232.18009766325" radius="0" d="M 425.4193548387097,57.819902336750005 h 18.870967741935484 v 232.18009766325 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="446.38709677419354" y="33.231202246750016" width="18.870967741935484" height="256.76879775325" radius="0" d="M 446.38709677419354,33.231202246750016 h 18.870967741935484 v 256.76879775325 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="467.35483870967744" y="48.191684057375" width="18.870967741935484" height="241.808315942625" radius="0" d="M 467.35483870967744,48.191684057375 h 18.870967741935484 v 241.808315942625 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="488.32258064516134" y="69.277817732125" width="18.870967741935484" height="220.72218226787498" radius="0" d="M 488.32258064516134,69.277817732125 h 18.870967741935484 v 220.72218226787498 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="509.2903225806452" y="103.320979197625" width="18.870967741935484" height="186.67902080237502" radius="0" d="M 509.2903225806452,103.320979197625 h 18.870967741935484 v 186.67902080237502 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="530.258064516129" y="156.11704249337498" width="18.870967741935484" height="133.88295750662502" radius="0" d="M 530.258064516129,156.11704249337498 h 18.870967741935484 v 133.88295750662502 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="551.2258064516129" y="149.72703694237498" width="18.870967741935484" height="140.27296305762502" radius="0" d="M 551.2258064516129,149.72703694237498 h 18.870967741935484 v 140.27296305762502 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="572.1935483870968" y="83.72563114825" width="18.870967741935484" height="206.27436885175" radius="0" d="M 572.1935483870968,83.72563114825 h 18.870967741935484 v 206.27436885175 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="593.1612903225807" y="82.62019729087498" width="18.870967741935484" height="207.37980270912502" radius="0" d="M 593.1612903225807,82.62019729087498 h 18.870967741935484 v 207.37980270912502 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="614.1290322580645" y="70.84724672112499" width="18.870967741935484" height="219.15275327887502" radius="0" d="M 614.1290322580645,70.84724672112499 h 18.870967741935484 v 219.15275327887502 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="635.0967741935484" y="83.59329985987499" width="18.870967741935484" height="206.406700140125" radius="0" d="M 635.0967741935484,83.59329985987499 h 18.870967741935484 v 206.406700140125 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="656.0645161290323" y="71.2411393915" width="18.870967741935484" height="218.75886060850002" radius="0" d="M 656.0645161290323,71.2411393915 h 18.870967741935484 v 218.75886060850002 h -18.870967741935484 Z"></path></g><g><path fill="#0088FE" name="undefined" x="677.0322580645161" y="223.01369346537498" width="18.870967741935484" height="66.98630653462502" radius="0" d="M 677.0322580645161,223.01369346537498 h 18.870967741935484 v 66.98630653462502 h -18.870967741935484 Z"></path></g></g></g></g><g id="recharts-bar-_r_1ah_"><g><g><g><path fill="#00C49F" name="undefined" x="48" y="86.404101239375" width="18.870967741935484" height="13.744881259750002" radius="0" d="M 48,86.404101239375 h 18.870967741935484 v 13.744881259750002 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="68.96774193548387" y="80.703402202" width="18.870967741935484" height="13.379107728874985" radius="0" d="M 68.96774193548387,80.703402202 h 18.870967741935484 v 13.379107728874985 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="89.93548387096774" y="158.35005343362502" width="18.870967741935484" height="8.424387160999999" radius="0" d="M 89.93548387096774,158.35005343362502 h 18.870967741935484 v 8.424387160999999 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="110.90322580645162" y="161.218908390625" width="18.870967741935484" height="8.489152835624992" radius="0" d="M 110.90322580645162,161.218908390625 h 18.870967741935484 v 8.489152835624992 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="131.8709677419355" y="95.57211930537501" width="18.870967741935484" height="14.133577884124975" radius="0" d="M 131.8709677419355,95.57211930537501 h 18.870967741935484 v 14.133577884124975 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="152.83870967741936" y="72.69720822512501" width="18.870967741935484" height="12.621155648750005" radius="0" d="M 152.83870967741936,72.69720822512501 h 18.870967741935484 v 12.621155648750005 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="173.80645161290323" y="57.010728622375" width="18.870967741935484" height="15.235428476124994" radius="0" d="M 173.80645161290323,57.010728622375 h 18.870967741935484 v 15.235428476124994 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="194.7741935483871" y="65.13070830425" width="18.870967741935484" height="17.428279484124985" radius="0" d="M 194.7741935483871,65.13070830425 h 18.870967741935484 v 17.428279484124985 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="215.74193548387098" y="64.91453865387498" width="18.870967741935484" height="17.39920836887501" radius="0" d="M 215.74193548387098,64.91453865387498 h 18.870967741935484 v 17.39920836887501 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="236.70967741935485" y="151.14389031937498" width="18.870967741935484" height="11.233006286875025" radius="0" d="M 236.70967741935485,151.14389031937498 h 18.870967741935484 v 11.233006286875025 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="257.6774193548387" y="162.64779540824998" width="18.870967741935484" height="11.50999212000005" radius="0" d="M 257.6774193548387,162.64779540824998 h 18.870967741935484 v 11.50999212000005 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="278.64516129032256" y="74.81646180787502" width="18.870967741935484" height="18.02387399849998" radius="0" d="M 278.64516129032256,74.81646180787502 h 18.870967741935484 v 18.02387399849998 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="299.61290322580646" y="56.10618769749999" width="18.870967741935484" height="15.189942860624996" radius="0" d="M 299.61290322580646,56.10618769749999 h 18.870967741935484 v 15.189942860624996 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="320.58064516129036" y="65.03616167049998" width="18.870967741935484" height="11.524288909750027" radius="0" d="M 320.58064516129036,65.03616167049998 h 18.870967741935484 v 11.524288909750027 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="341.5483870967742" y="67.7347998515" width="18.870967741935484" height="11.424890525249992" radius="0" d="M 341.5483870967742,67.7347998515 h 18.870967741935484 v 11.424890525249992 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="362.51612903225805" y="62.08185687587499" width="18.870967741935484" height="9.987376265000009" radius="0" d="M 362.51612903225805,62.08185687587499 h 18.870967741935484 v 9.987376265000009 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="383.48387096774195" y="114.253869642875" width="18.870967741935484" height="8.996351967875" radius="0" d="M 383.48387096774195,114.253869642875 h 18.870967741935484 v 8.996351967875 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="404.45161290322585" y="103.82912527412499" width="18.870967741935484" height="11.864857152250025" radius="0" d="M 404.45161290322585,103.82912527412499 h 18.870967741935484 v 11.864857152250025 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="425.4193548387097" y="44.62584316925" width="18.870967741935484" height="13.194059167500008" radius="0" d="M 425.4193548387097,44.62584316925 h 18.870967741935484 v 13.194059167500008 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="446.38709677419354" y="21.178932627125004" width="18.870967741935484" height="12.052269619625012" radius="0" d="M 446.38709677419354,21.178932627125004 h 18.870967741935484 v 12.052269619625012 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="467.35483870967744" y="36.141527207375006" width="18.870967741935484" height="12.050156849999993" radius="0" d="M 467.35483870967744,36.141527207375006 h 18.870967741935484 v 12.050156849999993 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="488.32258064516134" y="58.020778532625016" width="18.870967741935484" height="11.257039199499985" radius="0" d="M 488.32258064516134,58.020778532625016 h 18.870967741935484 v 11.257039199499985 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="509.2903225806452" y="92.16980803975001" width="18.870967741935484" height="11.151171157874984" radius="0" d="M 509.2903225806452,92.16980803975001 h 18.870967741935484 v 11.151171157874984 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="530.258064516129" y="146.121486930625" width="18.870967741935484" height="9.995555562749985" radius="0" d="M 530.258064516129,146.121486930625 h 18.870967741935484 v 9.995555562749985 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="551.2258064516129" y="138.42850667075" width="18.870967741935484" height="11.29853027162497" radius="0" d="M 551.2258064516129,138.42850667075 h 18.870967741935484 v 11.29853027162497 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="572.1935483870968" y="71.45803429200001" width="18.870967741935484" height="12.267596856249995" radius="0" d="M 572.1935483870968,71.45803429200001 h 18.870967741935484 v 12.267596856249995 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="593.1612903225807" y="71.08056690262498" width="18.870967741935484" height="11.539630388250004" radius="0" d="M 593.1612903225807,71.08056690262498 h 18.870967741935484 v 11.539630388250004 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="614.1290322580645" y="57.7794565805" width="18.870967741935484" height="13.067790140624986" radius="0" d="M 614.1290322580645,57.7794565805 h 18.870967741935484 v 13.067790140624986 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="635.0967741935484" y="70.75798958225002" width="18.870967741935484" height="12.83531027762497" radius="0" d="M 635.0967741935484,70.75798958225002 h 18.870967741935484 v 12.83531027762497 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="656.0645161290323" y="55.48378486900001" width="18.870967741935484" height="15.757354522499988" radius="0" d="M 656.0645161290323,55.48378486900001 h 18.870967741935484 v 15.757354522499988 h -18.870967741935484 Z"></path></g><g><path fill="#00C49F" name="undefined" x="677.0322580645161" y="216.936693592625" width="18.870967741935484" height="6.07699987274998" radius="0" d="M 677.0322580645161,216.936693592625 h 18.870967741935484 v 6.07699987274998 h -18.870967741935484 Z"></path></g></g></g></g><g id="recharts-bar-_r_1ai_"><g><g><g><path fill="#FFBB28" name="undefined" x="48" y="81.21248441300001" width="18.870967741935484" height="5.1916168263749825" radius="0" d="M 48,81.21248441300001 h 18.870967741935484 v 5.1916168263749825 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="68.96774193548387" y="75.154026549375" width="18.870967741935484" height="5.549375652625002" radius="0" d="M 68.96774193548387,75.154026549375 h 18.870967741935484 v 5.549375652625002 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="89.93548387096774" y="154.68876219199998" width="18.870967741935484" height="3.6612912416250367" radius="0" d="M 89.93548387096774,154.68876219199998 h 18.870967741935484 v 3.6612912416250367 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="110.90322580645162" y="158.20810171787502" width="18.870967741935484" height="3.0108066727499647" radius="0" d="M 110.90322580645162,158.20810171787502 h 18.870967741935484 v 3.0108066727499647 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="131.8709677419355" y="90.472948131625" width="18.870967741935484" height="5.099171173750008" radius="0" d="M 131.8709677419355,90.472948131625 h 18.870967741935484 v 5.099171173750008 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="152.83870967741936" y="67.375417775625" width="18.870967741935484" height="5.321790449500014" radius="0" d="M 152.83870967741936,67.375417775625 h 18.870967741935484 v 5.321790449500014 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="173.80645161290323" y="51.54808822150001" width="18.870967741935484" height="5.462640400874989" radius="0" d="M 173.80645161290323,51.54808822150001 h 18.870967741935484 v 5.462640400874989 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="194.7741935483871" y="59.833776256000014" width="18.870967741935484" height="5.296932048249992" radius="0" d="M 194.7741935483871,59.833776256000014 h 18.870967741935484 v 5.296932048249992 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="215.74193548387098" y="59.408178719374995" width="18.870967741935484" height="5.50635993449999" radius="0" d="M 215.74193548387098,59.408178719374995 h 18.870967741935484 v 5.50635993449999 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="236.70967741935485" y="147.883615532875" width="18.870967741935484" height="3.260274786499963" radius="0" d="M 236.70967741935485,147.883615532875 h 18.870967741935484 v 3.260274786499963 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="257.6774193548387" y="159.595633339875" width="18.870967741935484" height="3.052162068374969" radius="0" d="M 257.6774193548387,159.595633339875 h 18.870967741935484 v 3.052162068374969 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="278.64516129032256" y="70.01978368187498" width="18.870967741935484" height="4.796678126000032" radius="0" d="M 278.64516129032256,70.01978368187498 h 18.870967741935484 v 4.796678126000032 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="299.61290322580646" y="51.339143119999996" width="18.870967741935484" height="4.767044577499995" radius="0" d="M 299.61290322580646,51.339143119999996 h 18.870967741935484 v 4.767044577499995 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="320.58064516129036" y="60.426237875" width="18.870967741935484" height="4.609923795499981" radius="0" d="M 320.58064516129036,60.426237875 h 18.870967741935484 v 4.609923795499981 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="341.5483870967742" y="63.17785666350001" width="18.870967741935484" height="4.556943187999991" radius="0" d="M 341.5483870967742,63.17785666350001 h 18.870967741935484 v 4.556943187999991 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="362.51612903225805" y="57.70897766662501" width="18.870967741935484" height="4.37287920924998" radius="0" d="M 362.51612903225805,57.70897766662501 h 18.870967741935484 v 4.37287920924998 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="383.48387096774195" y="109.91419420112499" width="18.870967741935484" height="4.339675441750018" radius="0" d="M 383.48387096774195,109.91419420112499 h 18.870967741935484 v 4.339675441750018 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="404.45161290322585" y="99.454830437125" width="18.870967741935484" height="4.374294836999994" radius="0" d="M 404.45161290322585,99.454830437125 h 18.870967741935484 v 4.374294836999994 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="425.4193548387097" y="39.63640289149999" width="18.870967741935484" height="4.989440277750006" radius="0" d="M 425.4193548387097,39.63640289149999 h 18.870967741935484 v 4.989440277750006 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="446.38709677419354" y="15.858420287374985" width="18.870967741935484" height="5.320512339750019" radius="0" d="M 446.38709677419354,15.858420287374985 h 18.870967741935484 v 5.320512339750019 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="467.35483870967744" y="29.99924310012501" width="18.870967741935484" height="6.142284107249996" radius="0" d="M 467.35483870967744,29.99924310012501 h 18.870967741935484 v 6.142284107249996 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="488.32258064516134" y="52.135999845625" width="18.870967741935484" height="5.884778687000015" radius="0" d="M 488.32258064516134,52.135999845625 h 18.870967741935484 v 5.884778687000015 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="509.2903225806452" y="87.373915650625" width="18.870967741935484" height="4.795892389125015" radius="0" d="M 509.2903225806452,87.373915650625 h 18.870967741935484 v 4.795892389125015 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="530.258064516129" y="142.40046062262502" width="18.870967741935484" height="3.7210263079999777" radius="0" d="M 530.258064516129,142.40046062262502 h 18.870967741935484 v 3.7210263079999777 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="551.2258064516129" y="134.94567430962502" width="18.870967741935484" height="3.4828323611249914" radius="0" d="M 551.2258064516129,134.94567430962502 h 18.870967741935484 v 3.4828323611249914 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="572.1935483870968" y="65.49077460974999" width="18.870967741935484" height="5.967259682250017" radius="0" d="M 572.1935483870968,65.49077460974999 h 18.870967741935484 v 5.967259682250017 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="593.1612903225807" y="65.474097257125" width="18.870967741935484" height="5.606469645499985" radius="0" d="M 593.1612903225807,65.474097257125 h 18.870967741935484 v 5.606469645499985 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="614.1290322580645" y="52.686203839124985" width="18.870967741935484" height="5.093252741375018" radius="0" d="M 614.1290322580645,52.686203839124985 h 18.870967741935484 v 5.093252741375018 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="635.0967741935484" y="66.010710049" width="18.870967741935484" height="4.747279533250023" radius="0" d="M 635.0967741935484,66.010710049 h 18.870967741935484 v 4.747279533250023 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="656.0645161290323" y="50.16075634425002" width="18.870967741935484" height="5.3230285247499936" radius="0" d="M 656.0645161290323,50.16075634425002 h 18.870967741935484 v 5.3230285247499936 h -18.870967741935484 Z"></path></g><g><path fill="#FFBB28" name="undefined" x="677.0322580645161" y="215.067371347875" width="18.870967741935484" height="1.869322244750009" radius="0" d="M 677.0322580645161,215.067371347875 h 18.870967741935484 v 1.869322244750009 h -18.870967741935484 Z"></path></g></g></g></g><g id="recharts-bar-_r_1aj_"><g><g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g></g><g><path fill="url(#forecastStripes)" name="undefined" x="677.0322580645161" y="110.1616912349" width="18.870967741935484" height="104.90568011297499" radius="0" d="M 677.0322580645161,110.1616912349 h 18.870967741935484 v 104.90568011297499 h -18.870967741935484 Z"></path></g></g></g></g></svg>

Prompt

18.5B

Completion

1.68B

Reasoning

516M

Prompt tokens measure input size. Reasoning tokens show internal thinking before a response. Completion tokens reflect total output length.

## More models from[Nano Banana 2](https://openrouter.ai/google/gemini-3.1-flash-image)

[

Gemini 3.1 Flash Image, a.k.a. "Nano Banana 2," is Google’s latest state of the art image generation and editing model, delivering Pro-level visual quality at Flash speed. It combines advanced contextual understanding with fast, cost-efficient inference, making complex image generation and iterative edits significantly more accessible. Aspect ratios can be controlled with the image\_config API Parameter

](https://openrouter.ai/google/gemini-3.1-flash-image)[Nano Banana Pro](https://openrouter.ai/google/gemini-3-pro-image)

[

Nano Banana Pro is Google’s most advanced image-generation and editing model, built on Gemini 3 Pro. It extends the original Nano Banana with significantly improved multimodal reasoning, real-world grounding, and high-fidelity visual synthesis. The model generates context-rich graphics, from infographics and diagrams to cinematic composites, and can incorporate real-time information via Search grounding.

It offers industry-leading text rendering in images (including long passages and multilingual layouts), consistent multi-image blending, and accurate identity preservation across up to five subjects. Nano Banana Pro adds fine-grained creative controls such as localized edits, lighting and focus adjustments, camera transformations, and support for 2K/4K outputs and flexible aspect ratios. It is designed for professional-grade design, product visualization, storyboarding, and complex multi-element compositions while remaining efficient for general image creation workflows.

](https://openrouter.ai/google/gemini-3-pro-image)[Gemini Embedding 2](https://openrouter.ai/google/gemini-embedding-2)

[

Gemini Embedding 2 is Google's first multimodal embedding model. We currently support mapping text and images into a unified vector space for semantic search and retrieval-augmented generation (RAG). It supports input context up to 8,192 tokens and flexible output dimensions from 128 to 3,072 (recommended: 768, 1536, or 3,072). Designed for cross-modal similarity — you can embed a text query and retrieve the most relevant images, or vice versa — making it well-suited for multimodal search, recommendation, and document understanding pipelines.

](https://openrouter.ai/google/gemini-embedding-2)[Gemini 3.5 Flash](https://openrouter.ai/google/gemini-3.5-flash)

[

Gemini 3.5 Flash is Google's high-efficiency multimodal model, bringing near-Pro level coding and reasoning at Flash-tier cost and speed. It is highly optimized for coding proficiency and parallel agentic execution loops, supporting text, image, video, audio, and PDF inputs.

Defaults to medium thinking effort for faster and more cost-efficient responses, with full support for thinking levels (minimal, low, medium, high) for fine-grained cost/performance trade-offs.

](https://openrouter.ai/google/gemini-3.5-flash)[Chirp 3](https://openrouter.ai/google/chirp-3)

[

Chirp 3 is Google's latest multilingual speech-to-text model. It offers enhanced transcription accuracy across 24 GA languages and 77+ preview languages, with support for automatic language detection, automatic punctuation, and a built-in denoiser for cleaner audio processing.

](https://openrouter.ai/google/chirp-3)[Google Gemini Pro Latest](https://openrouter.ai/~google/gemini-pro-latest)

[

This model always redirects to the latest model in the Google Gemini Pro family.

](https://openrouter.ai/~google/gemini-pro-latest)[Google Gemini Flash Latest](https://openrouter.ai/~google/gemini-flash-latest)

[

This model always redirects to the latest model in the Google Gemini Flash family.

](https://openrouter.ai/~google/gemini-flash-latest)[Gemini 3.1 Flash TTS Preview](https://openrouter.ai/google/gemini-3.1-flash-tts-preview)

[

Gemini 3.1 Flash TTS Preview is a text-to-speech model from Google, and a substantial generational step up from Gemini 2.5 Flash TTS. It takes text input and produces audio output across 70+ languages — nearly 3× the language coverage of its predecessor.

The headline addition is a system of 200+ inline audio tags (e.g. `[whispers]`, `[laughs]`, `[excited]`) that let developers steer delivery, emotion, and pacing mid-sentence, alongside a "director's chair" workflow in Google AI Studio for defining per-character Audio Profiles and scene-level context. It supports up to two speakers with independent voice and style configuration per speaker, outputs PCM audio at 24 kHz / 16-bit mono, and automatically watermarks all output with SynthID. Context window is 32k tokens.

](https://openrouter.ai/google/gemini-3.1-flash-tts-preview)[Veo 3.1 Fast](https://openrouter.ai/google/veo-3.1-fast)

[

Google's mid-tier video generation model balancing speed and quality. Veo 3.1 Fast generates high-quality video from text or image prompts with native synchronized audio, offering faster turnaround than Veo 3.1 at lower cost. Supports first-frame and last-frame conditioning, multiple resolutions and aspect ratios, and SynthID watermarking.

](https://openrouter.ai/google/veo-3.1-fast)[Veo 3.1 Lite](https://openrouter.ai/google/veo-3.1-lite)

[

Google's most cost-effective video generation model, designed for high-volume applications and rapid iteration. Veo 3.1 Lite generates 720p and 1080p video from text or image prompts with native synchronized audio at less than 50% of the cost of Veo 3.1 Fast. Supports 4–8 second clips in landscape (16:9) and portrait (9:16) formats, with SynthID watermarking. Ideal for content platforms, short-form video creation, and automated media generation.

](https://openrouter.ai/google/veo-3.1-lite)[Gemini Embedding 2 Preview](https://openrouter.ai/google/gemini-embedding-2-preview)

[

Gemini Embedding 2 Preview is Google's first multimodal embedding model. We currently support mapping text and images into a unified vector space for semantic search and retrieval-augmented generation (RAG). It supports input context up to 8,192 tokens and flexible output dimensions from 128 to 3,072 (recommended: 768, 1536, or 3,072). Designed for cross-modal similarity — you can embed a text query and retrieve the most relevant images, or vice versa — making it well-suited for multimodal search, recommendation, and document understanding pipelines.

](https://openrouter.ai/google/gemini-embedding-2-preview)[Gemma 4 26B A4B](https://openrouter.ai/google/gemma-4-26b-a4b-it)

[

Gemma 4 26B A4B IT is an instruction-tuned Mixture-of-Experts (MoE) model from Google DeepMind. Despite 25.2B total parameters, only 3.8B activate per token during inference — delivering near-31B quality at a fraction of the compute cost. Supports multimodal input including text, images, and video (up to 60s at 1fps). Features a 256K token context window, native function calling, configurable thinking/reasoning mode, and structured output support. Released under Apache 2.0.

](https://openrouter.ai/google/gemma-4-26b-a4b-it)[Gemma 4 26B A4B](https://openrouter.ai/google/gemma-4-26b-a4b-it:free)

[

Gemma 4 26B A4B IT is an instruction-tuned Mixture-of-Experts (MoE) model from Google DeepMind. Despite 25.2B total parameters, only 3.8B activate per token during inference — delivering near-31B quality at a fraction of the compute cost. Supports multimodal input including text, images, and video (up to 60s at 1fps). Features a 256K token context window, native function calling, configurable thinking/reasoning mode, and structured output support. Released under Apache 2.0.

](https://openrouter.ai/google/gemma-4-26b-a4b-it:free)[Gemma 4 31B](https://openrouter.ai/google/gemma-4-31b-it)

[

Gemma 4 31B Instruct is Google DeepMind's 30.7B dense multimodal model supporting text and image input with text output. Features a 256K token context window, configurable thinking/reasoning mode, native function calling, and multilingual support across 140+ languages. Strong on coding, reasoning, and document understanding tasks. Apache 2.0 license.

](https://openrouter.ai/google/gemma-4-31b-it)[Gemma 4 31B](https://openrouter.ai/google/gemma-4-31b-it:free)

[

Gemma 4 31B Instruct is Google DeepMind's 30.7B dense multimodal model supporting text and image input with text output. Features a 256K token context window, configurable thinking/reasoning mode, native function calling, and multilingual support across 140+ languages. Strong on coding, reasoning, and document understanding tasks. Apache 2.0 license.

](https://openrouter.ai/google/gemma-4-31b-it:free)[Lyria 3 Pro Preview](https://openrouter.ai/google/lyria-3-pro-preview)

[

Full-length songs are priced at $0.08 per song. Lyria 3 is Google's family of music generation models, available through the Gemini API. With Lyria 3, you can generate high-quality, 48kHz stereo audio from text prompts or from images. These models deliver structural coherence, including vocals, timed lyrics, and full instrumental arrangements. Lyria 3 Pro can generate full-length songs with verses, choruses, bridges.

](https://openrouter.ai/google/lyria-3-pro-preview)[Lyria 3 Clip Preview](https://openrouter.ai/google/lyria-3-clip-preview)

[

30 second duration clips are priced at $0.04 per clip. Lyria 3 is Google's family of music generation models, available through the Gemini API. With Lyria 3, you can generate high-quality, 48kHz stereo audio from text prompts or from images. These models deliver structural coherence, including vocals, timed lyrics, and full instrumental arrangements. Lyria 3 Clip can generate short clips, loops, previews.

](https://openrouter.ai/google/lyria-3-clip-preview)[Veo 3.1](https://openrouter.ai/google/veo-3.1)

[

Google's state-of-the-art video generation model, built for maximum visual fidelity in final production cuts. Veo 3.1 generates high-quality 1080p video from text or image prompts with native synchronized audio — including dialogue, ambient effects, and background sound. Supports scene extension (up to 20 chained clips for 140+ second narratives), frames-to-video transitions between two images, vertical video for Shorts, and 4K upscaling.

](https://openrouter.ai/google/veo-3.1)[Gemini 3.1 Flash Lite Preview](https://openrouter.ai/google/gemini-3.1-flash-lite-preview)

[

Gemini 3.1 Flash Lite Preview is Google's high-efficiency model optimized for high-volume use cases. It outperforms Gemini 2.5 Flash Lite on overall quality and approaches Gemini 2.5 Flash performance across key capabilities. Improvements span audio input/ASR, RAG snippet ranking, translation, data extraction, and code completion. Supports full thinking levels (minimal, low, medium, high) for fine-grained cost/performance trade-offs. Priced at half the cost of Gemini 3 Flash.

](https://openrouter.ai/google/gemini-3.1-flash-lite-preview)[Nano Banana 2](https://openrouter.ai/google/gemini-3.1-flash-image-preview)

[

Gemini 3.1 Flash Image Preview, a.k.a. "Nano Banana 2," is Google’s latest state of the art image generation and editing model, delivering Pro-level visual quality at Flash speed. It combines advanced contextual understanding with fast, cost-efficient inference, making complex image generation and iterative edits significantly more accessible. Aspect ratios can be controlled with the image\_config API Parameter

](https://openrouter.ai/google/gemini-3.1-flash-image-preview)[Gemini 3.1 Pro Preview Custom Tools](https://openrouter.ai/google/gemini-3.1-pro-preview-customtools)

[

Gemini 3.1 Pro Preview Custom Tools is a variant of Gemini 3.1 Pro that improves tool selection behavior by preventing overuse of a general bash tool when more efficient third-party or user-defined functions are available. This specialized preview endpoint significantly increases function calling reliability and ensures the model selects the most appropriate tool in coding agents and complex, multi-tool workflows.

It retains the core strengths of Gemini 3.1 Pro, including multimodal reasoning across text, image, video, audio, and code, a 1M-token context window, and strong software engineering performance.

](https://openrouter.ai/google/gemini-3.1-pro-preview-customtools)[Gemini 3.1 Pro Preview](https://openrouter.ai/google/gemini-3.1-pro-preview)

[

Gemini 3.1 Pro Preview is Google’s frontier reasoning model, delivering enhanced software engineering performance, improved agentic reliability, and more efficient token usage across complex workflows. Building on the multimodal foundation of the Gemini 3 series, it combines high-precision reasoning across text, image, video, audio, and code with a 1M-token context window. Reasoning Details must be preserved when using multi-turn tool calling, see our docs here: https://openrouter.ai/docs/use-cases/reasoning-tokens#preserving-reasoning. The 3.1 update introduces measurable gains in SWE benchmarks and real-world coding environments, along with stronger autonomous task execution in structured domains such as finance and spreadsheet-based workflows.

Designed for advanced development and agentic systems, Gemini 3.1 Pro Preview improves long-horizon stability and tool orchestration while increasing token efficiency. It introduces a new medium thinking level to better balance cost, speed, and performance. The model excels in agentic coding, structured planning, multimodal analysis, and workflow automation, making it well-suited for autonomous agents, financial modeling, spreadsheet automation, and high-context enterprise tasks.

](https://openrouter.ai/google/gemini-3.1-pro-preview)[Gemini 3 Flash Preview](https://openrouter.ai/google/gemini-3-flash-preview)

[

Gemini 3 Flash Preview is a high speed, high value thinking model designed for agentic workflows, multi turn chat, and coding assistance. It delivers near Pro level reasoning and tool use performance with substantially lower latency than larger Gemini variants, making it well suited for interactive development, long running agent loops, and collaborative coding tasks. Compared to Gemini 2.5 Flash, it provides broad quality improvements across reasoning, multimodal understanding, and reliability.

The model supports a 1M token context window and multimodal inputs including text, images, audio, video, and PDFs, with text output. It includes configurable reasoning via thinking levels (minimal, low, medium, high), structured output, tool use, and automatic context caching. Gemini 3 Flash Preview is optimized for users who want strong reasoning and agentic behavior without the cost or latency of full scale frontier models.

](https://openrouter.ai/google/gemini-3-flash-preview)[Nano Banana Pro](https://openrouter.ai/google/gemini-3-pro-image-preview)

[

Nano Banana Pro is Google’s most advanced image-generation and editing model, built on Gemini 3 Pro. It extends the original Nano Banana with significantly improved multimodal reasoning, real-world grounding, and high-fidelity visual synthesis. The model generates context-rich graphics, from infographics and diagrams to cinematic composites, and can incorporate real-time information via Search grounding.

It offers industry-leading text rendering in images (including long passages and multilingual layouts), consistent multi-image blending, and accurate identity preservation across up to five subjects. Nano Banana Pro adds fine-grained creative controls such as localized edits, lighting and focus adjustments, camera transformations, and support for 2K/4K outputs and flexible aspect ratios. It is designed for professional-grade design, product visualization, storyboarding, and complex multi-element compositions while remaining efficient for general image creation workflows.

](https://openrouter.ai/google/gemini-3-pro-image-preview)[Gemini 3 Pro Preview](https://openrouter.ai/google/gemini-3-pro-preview)

[

Gemini 3 Pro is Google’s flagship frontier model for high-precision multimodal reasoning, combining strong performance across text, image, video, audio, and code with a 1M-token context window. Reasoning Details must be preserved when using multi-turn tool calling, see our docs here: https://openrouter.ai/docs/use-cases/reasoning-tokens#preserving-reasoning-blocks. It delivers state-of-the-art benchmark results in general reasoning, STEM problem solving, factual QA, and multimodal understanding, including leading scores on LMArena, GPQA Diamond, MathArena Apex, MMMU-Pro, and Video-MMMU. Interactions emphasize depth and interpretability: the model is designed to infer intent with minimal prompting and produce direct, insight-focused responses.

Built for advanced development and agentic workflows, Gemini 3 Pro provides robust tool-calling, long-horizon planning stability, and strong zero-shot generation for complex UI, visualization, and coding tasks. It excels at agentic coding (SWE-Bench Verified, Terminal-Bench 2.0), multimodal analysis, and structured long-form tasks such as research synthesis, planning, and interactive learning experiences. Suitable applications include autonomous agents, coding assistants, multimodal analytics, scientific reasoning, and high-context information processing.

](https://openrouter.ai/google/gemini-3-pro-preview)[Gemini Embedding 001](https://openrouter.ai/google/gemini-embedding-001)

[

gemini-embedding-001 provides a unified cutting edge experience across domains, including science, legal, finance, and coding. This embedding model has consistently held a top spot on the Massive Text Embedding Benchmark (MTEB) Multilingual leaderboard since the experimental launch in March.

](https://openrouter.ai/google/gemini-embedding-001)[Nano Banana](https://openrouter.ai/google/gemini-2.5-flash-image)

[

Gemini 2.5 Flash Image, a.k.a. "Nano Banana," is now generally available. It is a state of the art image generation model with contextual understanding. It is capable of image generation, edits, and multi-turn conversations. Aspect ratios can be controlled with the image\_config API Parameter

](https://openrouter.ai/google/gemini-2.5-flash-image)[Gemini 2.5 Flash Preview 09-2025](https://openrouter.ai/google/gemini-2.5-flash-preview-09-2025)

[

Gemini 2.5 Flash Preview September 2025 Checkpoint is Google's state-of-the-art workhorse model, specifically designed for advanced reasoning, coding, mathematics, and scientific tasks. It includes built-in "thinking" capabilities, enabling it to provide responses with greater accuracy and nuanced context handling.

Additionally, Gemini 2.5 Flash is configurable through the "max tokens for reasoning" parameter, as described in the documentation (https://openrouter.ai/docs/use-cases/reasoning-tokens#max-tokens-for-reasoning).

](https://openrouter.ai/google/gemini-2.5-flash-preview-09-2025)[Gemini 2.5 Flash Lite Preview 09-2025](https://openrouter.ai/google/gemini-2.5-flash-lite-preview-09-2025)

[

Gemini 2.5 Flash-Lite is a lightweight reasoning model in the Gemini 2.5 family, optimized for ultra-low latency and cost efficiency. It offers improved throughput, faster token generation, and better performance across common benchmarks compared to earlier Flash models. By default, "thinking" (i.e. multi-pass reasoning) is disabled to prioritize speed, but developers can enable it via the Reasoning API parameter to selectively trade off cost for intelligence.

](https://openrouter.ai/google/gemini-2.5-flash-lite-preview-09-2025)[Gemini 2.5 Flash Image Preview](https://openrouter.ai/google/gemini-2.5-flash-image-preview)

[

Gemini 2.5 Flash Image Preview, a.k.a. "Nano Banana," is a state of the art image generation model with contextual understanding. It is capable of image generation, edits, and multi-turn conversations.

](https://openrouter.ai/google/gemini-2.5-flash-image-preview)[Gemini 2.5 Flash Lite](https://openrouter.ai/google/gemini-2.5-flash-lite)

[

Gemini 2.5 Flash-Lite is a lightweight reasoning model in the Gemini 2.5 family, optimized for ultra-low latency and cost efficiency. It offers improved throughput, faster token generation, and better performance across common benchmarks compared to earlier Flash models. By default, "thinking" (i.e. multi-pass reasoning) is disabled to prioritize speed, but developers can enable it via the Reasoning API parameter to selectively trade off cost for intelligence.

](https://openrouter.ai/google/gemini-2.5-flash-lite)[Gemma 3n 2B](https://openrouter.ai/google/gemma-3n-e2b-it)

[

Gemma 3n E2B IT is a multimodal, instruction-tuned model developed by Google DeepMind, designed to operate efficiently at an effective parameter size of 2B while leveraging a 6B architecture. Based on the MatFormer architecture, it supports nested submodels and modular composition via the Mix-and-Match framework. Gemma 3n models are optimized for low-resource deployment, offering 32K context length and strong multilingual and reasoning performance across common benchmarks. This variant is trained on a diverse corpus including code, math, web, and multimodal data.

](https://openrouter.ai/google/gemma-3n-e2b-it)[Gemini 2.5 Flash](https://openrouter.ai/google/gemini-2.5-flash)

[

Gemini 2.5 Flash is Google's state-of-the-art workhorse model, specifically designed for advanced reasoning, coding, mathematics, and scientific tasks. It includes built-in "thinking" capabilities, enabling it to provide responses with greater accuracy and nuanced context handling.

Additionally, Gemini 2.5 Flash is configurable through the "max tokens for reasoning" parameter, as described in the documentation (https://openrouter.ai/docs/use-cases/reasoning-tokens#max-tokens-for-reasoning).

](https://openrouter.ai/google/gemini-2.5-flash)[Gemini 2.5 Pro](https://openrouter.ai/google/gemini-2.5-pro)

[

Gemini 2.5 Pro is Google’s state-of-the-art AI model designed for advanced reasoning, coding, mathematics, and scientific tasks. It employs “thinking” capabilities, enabling it to reason through responses with enhanced accuracy and nuanced context handling. Gemini 2.5 Pro achieves top-tier performance on multiple benchmarks, including first-place positioning on the LMArena leaderboard, reflecting superior human-preference alignment and complex problem-solving abilities.

](https://openrouter.ai/google/gemini-2.5-pro)[Gemini 2.5 Pro Preview 06-05](https://openrouter.ai/google/gemini-2.5-pro-preview)

[

Gemini 2.5 Pro is Google’s state-of-the-art AI model designed for advanced reasoning, coding, mathematics, and scientific tasks. It employs “thinking” capabilities, enabling it to reason through responses with enhanced accuracy and nuanced context handling. Gemini 2.5 Pro achieves top-tier performance on multiple benchmarks, including first-place positioning on the LMArena leaderboard, reflecting superior human-preference alignment and complex problem-solving abilities.

](https://openrouter.ai/google/gemini-2.5-pro-preview)[Gemma 1 2B](https://openrouter.ai/google/gemma-2b-it)

[

Gemma 1 2B by Google is an open model built from the same research and technology used to create the Gemini models.

Gemma models are well-suited for a variety of text generation tasks, including question answering, summarization, and reasoning.

Usage of Gemma is subject to Google's Gemma Terms of Use.

](https://openrouter.ai/google/gemma-2b-it)[Gemma 3n 4B](https://openrouter.ai/google/gemma-3n-e4b-it)

[

Gemma 3n E4B-it is optimized for efficient execution on mobile and low-resource devices, such as phones, laptops, and tablets. It supports multimodal inputs—including text, visual data, and audio—enabling diverse tasks such as text generation, speech recognition, translation, and image analysis. Leveraging innovations like Per-Layer Embedding (PLE) caching and the MatFormer architecture, Gemma 3n dynamically manages memory usage and computational load by selectively activating model parameters, significantly reducing runtime resource requirements.

This model supports a wide linguistic range (trained in over 140 languages) and features a flexible 32K token context window. Gemma 3n can selectively load parameters, optimizing memory and computational efficiency based on the task or device capabilities, making it well-suited for privacy-focused, offline-capable applications and on-device AI solutions. Read more in the blog post

](https://openrouter.ai/google/gemma-3n-e4b-it)[Gemini 2.5 Pro Preview 05-06](https://openrouter.ai/google/gemini-2.5-pro-preview-05-06)

[

Gemini 2.5 Pro is Google’s state-of-the-art AI model designed for advanced reasoning, coding, mathematics, and scientific tasks. It employs “thinking” capabilities, enabling it to reason through responses with enhanced accuracy and nuanced context handling. Gemini 2.5 Pro achieves top-tier performance on multiple benchmarks, including first-place positioning on the LMArena leaderboard, reflecting superior human-preference alignment and complex problem-solving abilities.

](https://openrouter.ai/google/gemini-2.5-pro-preview-05-06)[Gemini 2.5 Pro Experimental](https://openrouter.ai/google/gemini-2.5-pro-exp-03-25)

[

This model has been deprecated by Google in favor of the (paid Preview model)\[google/gemini-2.5-pro-preview\] Gemini 2.5 Pro is Google’s state-of-the-art AI model designed for advanced reasoning, coding, mathematics, and scientific tasks. It employs “thinking” capabilities, enabling it to reason through responses with enhanced accuracy and nuanced context handling. Gemini 2.5 Pro achieves top-tier performance on multiple benchmarks, including first-place positioning on the LMArena leaderboard, reflecting superior human-preference alignment and complex problem-solving abilities.

](https://openrouter.ai/google/gemini-2.5-pro-exp-03-25)[Gemma 3 1B](https://openrouter.ai/google/gemma-3-1b-it)

[

Gemma 3 1B is the smallest of the new Gemma 3 family. It handles context windows up to 32k tokens, understands over 140 languages, and offers improved math, reasoning, and chat capabilities, including structured outputs and function calling. Note: Gemma 3 1B is not multimodal. For the smallest multimodal Gemma 3 model, please see Gemma 3 4B

](https://openrouter.ai/google/gemma-3-1b-it)[Gemma 3 4B](https://openrouter.ai/google/gemma-3-4b-it)

[

Gemma 3 introduces multimodality, supporting vision-language input and text outputs. It handles context windows up to 128k tokens, understands over 140 languages, and offers improved math, reasoning, and chat capabilities, including structured outputs and function calling.

](https://openrouter.ai/google/gemma-3-4b-it)[Gemma 3 12B](https://openrouter.ai/google/gemma-3-12b-it)

[

Gemma 3 introduces multimodality, supporting vision-language input and text outputs. It handles context windows up to 128k tokens, understands over 140 languages, and offers improved math, reasoning, and chat capabilities, including structured outputs and function calling. Gemma 3 12B is the second largest in the family of Gemma 3 models after Gemma 3 27B

](https://openrouter.ai/google/gemma-3-12b-it)[Gemma 3 27B](https://openrouter.ai/google/gemma-3-27b-it)

[

Gemma 3 introduces multimodality, supporting vision-language input and text outputs. It handles context windows up to 128k tokens, understands over 140 languages, and offers improved math, reasoning, and chat capabilities, including structured outputs and function calling. Gemma 3 27B is Google's latest open source model, successor to Gemma 2

](https://openrouter.ai/google/gemma-3-27b-it)[Gemini 2.0 Flash Lite](https://openrouter.ai/google/gemini-2.0-flash-lite-001)

[

Gemini 2.0 Flash Lite offers a significantly faster time to first token (TTFT) compared to Gemini Flash 1.5, while maintaining quality on par with larger models like Gemini Pro 1.5, all at extremely economical token prices.

](https://openrouter.ai/google/gemini-2.0-flash-lite-001)[Gemini 2.0 Flash](https://openrouter.ai/google/gemini-2.0-flash-001)

[

Gemini Flash 2.0 offers a significantly faster time to first token (TTFT) compared to Gemini Flash 1.5, while maintaining quality on par with larger models like Gemini Pro 1.5. It introduces notable enhancements in multimodal understanding, coding capabilities, complex instruction following, and function calling. These advancements come together to deliver more seamless and robust agentic experiences.

](https://openrouter.ai/google/gemini-2.0-flash-001)[Gemini 2.0 Flash Experimental](https://openrouter.ai/google/gemini-2.0-flash-exp)

[

Gemini Flash 2.0 offers a significantly faster time to first token (TTFT) compared to Gemini Flash 1.5, while maintaining quality on par with larger models like Gemini Pro 1.5. It introduces notable enhancements in multimodal understanding, coding capabilities, complex instruction following, and function calling. These advancements come together to deliver more seamless and robust agentic experiences.

](https://openrouter.ai/google/gemini-2.0-flash-exp)[Gemini Experimental 1121](https://openrouter.ai/google/gemini-exp-1121)

[

Experimental release (November 21st, 2024) of Gemini.

](https://openrouter.ai/google/gemini-exp-1121)[Gemini Experimental 1114](https://openrouter.ai/google/gemini-exp-1114)

[

Gemini 11-14 (2024) experimental model features "quality" improvements.

](https://openrouter.ai/google/gemini-exp-1114)[Gemini 1.5 Flash 8B](https://openrouter.ai/google/gemini-flash-1.5-8b)

[

Gemini Flash 1.5 8B is optimized for speed and efficiency, offering enhanced performance in small prompt tasks like chat, transcription, and translation. With reduced latency, it is highly effective for real-time and large-scale operations. This model focuses on cost-effective solutions while maintaining high-quality results.

Click here to learn more about this model.

Usage of Gemini is subject to Google's Gemini Terms of Use.

](https://openrouter.ai/google/gemini-flash-1.5-8b)[Gemini 1.5 Flash Experimental](https://openrouter.ai/google/gemini-flash-1.5-exp)

[

Gemini 1.5 Flash Experimental is an experimental version of the Gemini 1.5 Flash model.

Usage of Gemini is subject to Google's Gemini Terms of Use.

#multimodal

Note: This model is experimental and not suited for production use-cases. It may be removed or redirected to another model in the future.

](https://openrouter.ai/google/gemini-flash-1.5-exp)[Gemini 1.5 Pro Experimental](https://openrouter.ai/google/gemini-pro-1.5-exp)

[

Gemini 1.5 Pro Experimental is a bleeding-edge version of the Gemini 1.5 Pro model. Because it's currently experimental, it will be **heavily rate-limited** by Google.

Usage of Gemini is subject to Google's Gemini Terms of Use.

#multimodal

](https://openrouter.ai/google/gemini-pro-1.5-exp)[Gemma 2 27B](https://openrouter.ai/google/gemma-2-27b-it)

[

Gemma 2 27B by Google is an open model built from the same research and technology used to create the Gemini models.

Gemma models are well-suited for a variety of text generation tasks, including question answering, summarization, and reasoning.

See the launch announcement for more details. Usage of Gemma is subject to Google's Gemma Terms of Use.

](https://openrouter.ai/google/gemma-2-27b-it)[Gemma 2 9B](https://openrouter.ai/google/gemma-2-9b-it)

[

Gemma 2 9B by Google is an advanced, open-source language model that sets a new standard for efficiency and performance in its size class.

Designed for a wide variety of tasks, it empowers developers and researchers to build innovative applications, while maintaining accessibility, safety, and cost-effectiveness.

See the launch announcement for more details. Usage of Gemma is subject to Google's Gemma Terms of Use.

](https://openrouter.ai/google/gemma-2-9b-it)[Gemini 1.5 Flash](https://openrouter.ai/google/gemini-flash-1.5)

[

Gemini 1.5 Flash is a foundation model that performs well at a variety of multimodal tasks such as visual understanding, classification, summarization, and creating content from image, audio and video. It's adept at processing visual and text inputs such as photographs, documents, infographics, and screenshots.

Gemini 1.5 Flash is designed for high-volume, high-frequency tasks where cost and latency matter. On most common tasks, Flash achieves comparable quality to other Gemini Pro models at a significantly reduced cost. Flash is well-suited for applications like chat assistants and on-demand content generation where speed and scale matter.

Usage of Gemini is subject to Google's Gemini Terms of Use.

#multimodal

](https://openrouter.ai/google/gemini-flash-1.5)[Gemini 1.5 Pro](https://openrouter.ai/google/gemini-pro-1.5)

[

Google's latest multimodal model, supports image and video\[0\] in text or chat prompts.

Optimized for language tasks including:

Usage of Gemini is subject to Google's Gemini Terms of Use.

](https://openrouter.ai/google/gemini-pro-1.5)[Gemma 7B](https://openrouter.ai/google/gemma-7b-it)

[

Gemma by Google is an advanced, open-source language model family, leveraging the latest in decoder-only, text-to-text technology. It offers English language capabilities across text generation tasks like question answering, summarization, and reasoning. The Gemma 7B variant is comparable in performance to leading open source models.

Usage of Gemma is subject to Google's Gemma Terms of Use.

](https://openrouter.ai/google/gemma-7b-it)[PaLM 2 Chat 32k](https://openrouter.ai/google/palm-2-chat-bison-32k)

[

PaLM 2 is a language model by Google with improved multilingual, reasoning and coding capabilities.

](https://openrouter.ai/google/palm-2-chat-bison-32k)[PaLM 2 Code Chat 32k](https://openrouter.ai/google/palm-2-codechat-bison-32k)

[

PaLM 2 fine-tuned for chatbot conversations that help with code-related questions.

](https://openrouter.ai/google/palm-2-codechat-bison-32k)[PaLM 2 Chat](https://openrouter.ai/google/palm-2-chat-bison)

[

PaLM 2 is a language model by Google with improved multilingual, reasoning and coding capabilities.

](https://openrouter.ai/google/palm-2-chat-bison)[PaLM 2 Code Chat](https://openrouter.ai/google/palm-2-codechat-bison)

[

PaLM 2 fine-tuned for chatbot conversations that help with code-related questions.

](https://openrouter.ai/google/palm-2-codechat-bison)