---
title: "Payments"
description: "Make payments to superluminar"
---

# Make a payment to superluminar!
<div class="col-4 p2 border-box border rounded">
    <p class="h3">Silver Sponsorship</p>
    <button class="btn white bg-blue rounded" id="checkout-button-silver">Pay 500 €</button>
</div>
<div class="p2 border-box">
</div>
<div class="col-4 p2 border-box border rounded">
    <p class="h3">Gold Sponsorship</p>
    <button class="btn white bg-blue rounded" id="checkout-button-gold">Pay 1000 €</button>
</div>

<script src="https://js.stripe.com/v3"></script>
<script>
var stripe = Stripe('pk_test_XDvvPIsJ65tSpBxMGKWEbl2b', {
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
    .getElementById('checkout-button-silver')
    .addEventListener('click', eventListenerProducer('sku_E9WruwOlSqOBPI'));
document
    .getElementById('checkout-button-silver')
    .addEventListener('click', eventListenerProducer('sku_E9WrDzWq2Z80QN'));
</script>