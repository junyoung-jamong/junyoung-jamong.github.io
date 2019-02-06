---
layout: post
title:  "Tesseract OCR Engine 개요"
date:   2019-02-06 16:45:00 +0900
categories: Computer vision, OCR
---

## Architecture

The first step is a connected component analysis in which outlines of the components are stored.

Recognition then proceeds as a two-pass process. In the first pass, an attempt is made to recognize each word in turn. Each word that is satisfactory is passed to an adaptive classifier as training data. The adaptive classifier then gets a chance to more accurately recognize text lower down the page.

A final phase resolves fuzzy spaces, and checks alternative hypotheses for the x-height to locate smallcap text.

### Line And word Finding

### Word Recognition

### Classification


[An Overview of the Tesseract OCR Engine](https://github.com/tesseract-ocr/docs/blob/master/tesseracticdar2007.pdf)