---
title: "Payments"
description: "Make payments to superluminar"
---

# Make a payment to superluminar!

<!-- Load Stripe.js on your website. -->
<script src="https://js.stripe.com/v3"></script>

<!-- Create a button that your customers click to complete their purchase. -->

<div class="product">
    <h4>Silver Sponsorship</h4>
    <button id="checkout-button-silver">Pay 500 €</button>
</div>

<div class="product">
    <h4>Gold Sponsorship</h4>
    <button id="checkout-button-gold">Pay 1000 €</button>
</div>

<script>
    var stripe = Stripe('pk_test_XDvvPIsJ65tSpBxMGKWEbl2b', {
      betas: ['checkout_beta_4']
    });

    var checkoutButtonSilver = document.getElementById('checkout-button-silver');
    var checkoutButtonGold = document.getElementById('checkout-button-gold');

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
    }
    checkoutButtonSilver.addEventListener('click', eventListenerProducer('sku_E9WruwOlSqOBPI'));
    checkoutButtonGold.addEventListener('click', eventListenerProducer('sku_E9WrDzWq2Z80QN'));
</script>