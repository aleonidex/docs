Let's assume that 115,000 minutes of audio were recognized and analyzed during one month of using {{ speechsense-name }}

**Recognition cost**

> {% calc [currency=USD] {{ sku|USD|speechsense.analysis.audio|number }} × 60 %} × 100,000 + {% calc [currency=USD] {{ sku|USD|speechsense.analysis.audio|pricingRate.6000000|number }} × 60 %} × 15,000 = {% calc [currency=USD] {{ sku|USD|speechsense.analysis.audio|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.audio|pricingRate.6000000|number }} × 60 × 15000 %}
>
> Total: {% calc [currency=USD] {{ sku|USD|speechsense.analysis.audio|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.audio|pricingRate.6000000|number }} × 60 × 15000 %}: Cost of recognizing 115,000 minutes of two-channel audio files.

Where:
* {% calc [currency=USD] {{ sku|USD|speechsense.analysis.audio|number }} × 60 %}: Cost per minute for the first 100,000 minutes.
* {% calc [currency=USD] {{ sku|USD|speechsense.analysis.audio|pricingRate.6000000|number }} × 60 %}: Cost per minute for the next 15,000 minutes.

**Analysis cost**

> {% calc [currency=USD] {{ sku|USD|speechsense.analysis.v1|number }} × 60 %} × 100,000 + {% calc [currency=USD] {{ sku|USD|speechsense.analysis.v1|pricingRate.6000000|number }} × 60 %} × 15,000 = {% calc [currency=USD] {{ sku|USD|speechsense.analysis.v1|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.v1|pricingRate.6000000|number }} × 60 × 15000 %}
>
> Total: {% calc [currency=USD] {{ sku|USD|speechsense.analysis.v1|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.v1|pricingRate.6000000|number }} × 60 × 15000 %}: Cost of analyzing 115,000 minutes of two-channel audio files.

Where:
* {% calc [currency=USD] {{ sku|USD|speechsense.analysis.v1|number }} × 60 %}: Cost per minute for the first 100,000 minutes.
* {% calc [currency=USD] {{ sku|USD|speechsense.analysis.v1|pricingRate.6000000|number }} × 60 %}: Cost per minute for the next 15,000 minutes.

**Total cost**

> {% calc [currency=USD] {{ sku|USD|speechsense.analysis.audio|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.audio|pricingRate.6000000|number }} × 60 × 15 000 %} + {% calc [currency=USD] {{ sku|USD|speechsense.analysis.v1|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.v1|pricingRate.6000000|number }} × 60 × 15 000 %} = {% calc [currency=USD] {{ sku|USD|speechsense.analysis.audio|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.audio|pricingRate.6000000|number }} × 60 × 15000 + {{ sku|USD|speechsense.analysis.v1|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.v1|pricingRate.6000000|number }} × 60 × 15000 %}
>
> Total: {% calc [currency=USD] {{ sku|USD|speechsense.analysis.audio|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.audio|pricingRate.6000000|number }} × 60 × 15000 + {{ sku|USD|speechsense.analysis.v1|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.v1|pricingRate.6000000|number }} × 60 × 15 000 %}: Cost of 115,000 minutes of using {{ speechsense-name }}.

Where:
* {% calc [currency=USD] {{ sku|USD|speechsense.analysis.audio|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.audio|pricingRate.6000000|number }} × 60 × 15000 %}: Cost of recognizing 115,000 minutes of two-channel audio files.
* {% calc [currency=USD] {{ sku|USD|speechsense.analysis.v1|number }} × 60 × 100000 + {{ sku|USD|speechsense.analysis.v1|pricingRate.6000000|number }} × 60 × 15000 %}: Cost of analyzing 115,000 minutes of two-channel audio files.
