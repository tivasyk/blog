---
layout: post
title:  "примітивний калькулятор на javascript'ах"
date:   2020-03-10 14:40
categories: blog
tags: 
  - щоденник
  - навчання
  - програмування
  - javascript
  - робота
  - комп'ютери
---

на [новій роботі]({% post_url 2020/2020-02-07-pfsense-proxmox %}) оплатили мені [курс javascript на edx'і](https://courses.edx.org/courses/course-v1:PennX+SD4x+3T2019) — знову навчаюся, потихеньку. пробігли галопом засади hmtl/css, вступ до javascript, експорт/імпорт json'ів, використання jquery… аж от чергове (третє) домашнє завдання — примітивний *калькулятор на чотири дії*.

основу накидав за півгодини — два регістри операндів, операція, прапорець очікування нового операнда, — а тоді ще два дні мізкував над кількома специфічними тестами автооцінювача (edx використовує [codio](https://codio.com) для цього курсу). але нічого вимудрував (кому цікаво: код під ілюстрацією, критикуйте; html/css не додаватиму).

навіщо компанія оплатила курс? хочуть, щоби я допомагав із запровадженням нової системи erp; може знадобитися (читай: майже гарантовано знадобиться) щось «допилювати». навіщо javascript мені особисто? бо цікаво, — і, може, колись ще покращуватиму щоденничок… хоча що ближче до пенсії, то частіше зміст повністю і розгромно перемагає форму.

![зображення](/assets/images/2020/2020-03-10-javascript-calc.jpg)

    // Constants
    var maxDisplay = 10; // The max length of the number on display

    // Internal variables
    var registerY = "";   // The "hidden" operand
    var registerX = "";   // The operand on the display
    var Operation = null; // The operation (+-*/)
    var waitingOperand = false; // Calculator state

    function log() {
        console.log("[" + String(registerY) + "](" + Operation + ")[" + String(registerX) + "] (" + String(waitingOperand ? "*" : " ") + ")");
    }

    function reset() {
        registerY = "";
        registerX = "";
        Operation = null;
        waitingOperand = true;
        updateDisplay();
        log();
    }

    function updateDisplay() {
        $("input#display").val(String(registerX));
    }

    function enterDigit(digit) {
        if (registerX.length < maxDisplay)  {
            if (waitingOperand) {
                registerY = registerX;
            }
            if (registerX == "0" || registerX == "" || waitingOperand) {
                registerX = String(digit);
            }
            else {
                registerX = registerX + String(digit);
            }
            waitingOperand = false;
            updateDisplay();
        }
    }

    function enterOperation(operation) {
        if (registerY != "" && Operation != null && waitingOperand == false) {
            Calculate();
        }
        Operation = operation;
        waitingOperand = true;
    }

    function Calculate() {
        if (registerY != "") {
            var temp = registerX;
            switch (Operation) {
                case "add":
                    if (waitingOperand) {
                        registerX = String(eval(registerX) + eval(registerY));
                    }
                    else {
                        registerX = String(eval(registerY) + eval(registerX));
                        registerY = temp;
                    }
                    break;
                case "sub":
                    if (waitingOperand)  {
                        registerX = String(eval(registerX) - eval(registerY));
                    } 
                    else {
                        registerX = String(eval(registerY) - eval(registerX));
                        registerY = temp;
                    }
                    break;
                case "mul":
                    if (waitingOperand) {
                        registerX = String(eval(registerX) * eval(registerY));
                    }
                    else {
                        registerX = String(eval(registerY) * eval(registerX));
                        registerY = temp;
                    }
                    break;
                case "div":
                    if (waitingOperand) {
                        registerX = String(eval(registerX) / eval(registerY));
                    }
                    else {
                        registerX = String(eval(registerY) / eval(registerX));
                        registerY = temp;
                    }
                    break;
                default:
                    break;
            }
        }
        waitingOperand = true;
        updateDisplay();
    }

    // Handle digit buttons
    $("document").ready(reset);
    $("#clearButton").click(reset);
    $("button.digitButton").click(function () {
        enterDigit($(this).val());
        log();
    });
    $("button.operationButton").click(function() {
        switch ($(this).attr("id")) {
            case "addButton":
                enterOperation("add");
                break;
            case "subtractButton":
                enterOperation("sub");
                break;
            case "multiplyButton":
                enterOperation("mul");
                break;
            case "divideButton":
                enterOperation("div");
                break;
            default:
                Operation = null;
                break;
        }
        waitingOperand = true;
        log();
    });
    $("#equalsButton").click(function() {
        Calculate();
        log();
    });
