---
title: "Payments"
description: "Make payments to superluminar"
---

# Make a payment to superluminar!
<div class="col-4 p2 border-box border rounded">
    <p class="h3">Diversity Sponsor - Adobe</p>
    <button class="btn white bg-blue rounded" id="checkout-button-adobe">Pay 1487,50 €</button>
</div>

<script src="https://js.stripe.com/v3"></script>
<script>
var stripe = Stripe('pk_live_gHUNN1o0YoOOfWhuhEdNjiFk', {
    betas: ['checkout_beta_4']
});

var eventListenerProducer = function(sku) {
    return function () {
        stripe.redirectToCheckout({
            items: [{sku: sku, quantity: 1}],
            successUrl: 'https://superluminar.io',
            cancelUrl: 'https://superluminar.io',
        })
            .then(function (result) {
                if (result.error) {
                    var displayError = document.getElementById('error-message');
                    displayError.textContent = result.error.message;
                }
            });
    };
};

document
    .getElementById('checkout-button-adobe')
    .addEventListener('click', eventListenerProducer('sku_EHMdIBl9y2t6t5'));
</script>