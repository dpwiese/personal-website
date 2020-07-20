---
title: "CRC Calculator"
date: 2020-07-18T17:49:17-05:00
draft: false
---

<link rel="stylesheet" href="style.css" />

<div class="page-container">
  <div class="form-container">
    <form class="form">
      <h1 class="title">CRC-32 Calculator</h1>
      <div class="label-container">
        <p>Enter payload in Hex:</p>
      </div>
      <div class="input-container">
        <input id="messageInput" type="text" name="Message" class="input">
      </div>
      <div class="button-container">
        <button type="submit" class="button">
          <span>Calculate</span>
        </button>
      </div>
      <div class="output-container">
        <p>CRC-32 (little endian): <span id="outputLittle"></span></p>
      </div>
      <div class="output-container">
        <p>CRC-32 (big endian): <span id="outputBig"></span></p>
      </div>
    </form>
  </div>
</div>

<script src="crc-utils.js"></script>

<script type="text/javascript">
  window.onload = function() {
    // Get query parameters
    const urlParams = new URLSearchParams(window.location.search);
    const messageValue = urlParams.get('Message');

    // Calculate CRC-32
    const hexString = conditionHexString(messageValue);
    const outputValueBig = calcCrc32(hexString).toString(16).toUpperCase();
    const outputValueLittle = changeEndianness(outputValueBig);

    // Set form input values
    const messageInput = document.getElementById("messageInput");
    messageInput.value = messageValue;

    // Set output values
    const outputLittle = document.getElementById("outputLittle");
    const outputBig = document.getElementById("outputBig");
    outputLittle.innerHTML = outputValueLittle;
    outputBig.innerHTML = outputValueBig;
  };
</script>
