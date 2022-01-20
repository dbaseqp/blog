---
layout: landing
titles:
  # @start locale config
  en      : &EN       About Me
  en-GB   : *EN
  en-US   : *EN
  en-CA   : *EN
  en-AU   : *EN
  zh-Hans : &ZH_HANS  关于我
  zh      : *ZH_HANS
  zh-CN   : *ZH_HANS
  zh-SG   : *ZH_HANS
  zh-Hant : &ZH_HANT  關於我
  zh-TW   : *ZH_HANT
  zh-HK   : *ZH_HANT
  ko      : &KO       소개
  ko-KR   : *KO
  fr      : &FR       À propos
  fr-BE   : *FR
  fr-CA   : *FR
  fr-CH   : *FR
  fr-FR   : *FR
  fr-LU   : *FR
  # @end locale config
key: page-about
#full_width: false
header: true

article_header:
  height: 100vh
  theme: dark
  background_color: "#367a9a"
  background_image:
    gradient: "linear-gradient(rgba(0, 0, 0, .2), rgba(0, 0, 0, .6))"
    src: /assets/images/logo/logo.png
data:
  sections:
    - title: Gabriel Fok <i class="fas fa-shield-alt"></i>
      excerpt: >
        Also known as baseq.
        <br>
        I'm a security enthusiest, avid esports fan, and learner of just about anything I find interesting.
        <br>
        I represent my university at security competitions as the CCDC Linux Security lead and a member of the CPTC team. 
        <br>
        I like learning to spread my knowledge and experience.
        <br>
        Current director of SWIFT Academy.
        <br>
        <br>
        Currently a Computer Science Undergraduate at Cal Poly Pomona. <i class="fas fa-horse-head"></i>
      actions:
        - text: Dotfiles
          url: https://github.com/dbaseqp/dotfiles
    - title: My Certifications <i class="fas fa-graduation-cap"></i>
      excerpt: >
        CompTIA A+
        <br>
---
## <center> Achievements <i class="fas fa-trophy"></i> </center>
<style>
  .swiper-demo {
    height: 220px;
  }
  .swiper-demo .swiper__slide {
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 3rem;
    color: #fff;
  }
  .swiper-demo .swiper__slide:nth-child(even) {
    background-color: #ff69b4;
  }
  .swiper-demo .swiper__slide:nth-child(odd) {
    background-color: #2593fc;
  }
  .swiper-demo--dark .swiper__slide:nth-child(even) {
    background-color: #C34160;
  }
  .swiper-demo--dark .swiper__slide:nth-child(odd) {
    background-color: #3267b6;
  }
  .swiper-demo--dark .swiper__slide:nth-child(1) {
    background-image: url("assets/images/logos/CPTCLogo_FullColorWithText_medium.png");
    background-size: 396px 230px;
    background-repeat: no-repeat;
  }
  .swiper-demo--dark .swiper__slide:nth-child(2) {
    background-image: url("assets/images/logos/CCDCLogo_FullColorWithText.png");
    background-size: 450px 216px;
    background-repeat: no-repeat;
  }
  .swiper-demo--dark .swiper__slide:nth-child(3) {
    background-image: url("assets/images/logos/CyberPatriotLogo_FullColor.png");
    background-size: 210px 200px;
    background-repeat: no-repeat;
  }
  .swiper-demo--image .swiper__slide:nth-child(n) {
    background-color: #000;
  }
</style>

<div class="swiper my-3 swiper-demo swiper-demo--dark swiper-achievements">
  <div class="swiper__wrapper">
    <div class="swiper__slide">1st Place | CPTC World Finals | 2021-2022<br>
    1st Place | CPTC Western Regionals  | 2021-2022</div>
    <div class="swiper__slide">9th Place | CCDC Western Qualifiers | 2020-2021
    </div>
    <div class="swiper__slide">4th Place | CyberPatriot Nationals | 2019-2020<br>
    6th Place | CyberPatriot Nationals | 2018-2019</div>
  </div>
  <!-- <div class="swiper__pagination"></div> -->
  <div class="swiper__button swiper__button--prev fas fa-chevron-left"></div>
  <div class="swiper__button swiper__button--next fas fa-chevron-right"></div>
  <!-- <div class="swiper-scrollbar"></div> -->
</div>

<script>
  {%- include scripts/lib/swiper.js -%}
  var SOURCES = window.TEXT_VARIABLES.sources;
  window.Lazyload.js(SOURCES.jquery, function() {
    $('.swiper-achievements').swiper();
  });
</script>