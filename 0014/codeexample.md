## Specification, Engineering, Schematics and Prototype
Game requirements:
* Any SDK can be used, but better chose those, that is being supported by most of platforms.
* Logics and Visual content is not crucial, that is why it can be ignored
* Game should work online, based on JS. Because our system is adopted for players, who comes from the web
* Gambling logic - player pays for each shot and can win more money
* Game should use our PRNG with our bankroll

Due to a short time frame, we going to use any existing or modified examples, like this one https://aframe.io/blog/

Schematics and detalised graphics is not crucial, that is why  we decided to take an example with pool

* In the center of the screen there is a dot, which is a sight
* By pressing any button (analog of tap and controller button push) shot transaction is being executed. Flash appears on the screen
* In case if an object was in the sight, logic.js transaction executes with function “shot”
* “Shot” function has a 50% chance of hitting the target. In case of hitting player receives one score, in other case - he loses one score.
* By closing the channel, game ends

Here are code examples on WebVR. In case of pointing the sight to an object, “sight” flag sets to 1.

```
AFRAME.registerComponent('change-color-on-hover', {
  schema: {
    color: {
      default: 'red'
    }
  },
  init: function() {
    var data = this.data;
    var el = this.el;

    var defaultColor = el.getAttribute('material').color;
    el.addEventListener('mouseenter', function() {
      el.setAttribute('color', data.color);
      sight = 1;

    });
    el.addEventListener('mouseleave', function() {
      el.setAttribute('color', defaultColor);
      sight = 0;
    });
  }
});
```
In case of shot, “shot” function executes. It requests bankroll through DCLib App.call

```
function shoot() {
  var randomSeed = DCLib.Utils.makeSeed();
  App.call('maketx', [1, 'confirm(' + randomSeed + ')'], function(result) {
    AFRAME.utils.entity.setComponentProperty(document.querySelector('#score'), 'text.value', "score \n" + App.logic.payChannel.getBalance());
    $("#whitepano").show().fadeOut(500);
  });
}

document.body.onkeyup = function(e) {
  if (e.keyCode == 32) {
    shoot();
  }
}

window.addEventListener('touchstart', function(e) {
  shoot();
});
```

Here is logic.js 
```
DCLib.defineDAppLogic('vrducks', function(){
  const _self = this

  var maketx = function(n,random_hash){
    const random_num = DCLib.numFromHash(random_hash, 0, 1);
    if (random_num == 1) _self.payChannel.addTX( 1 )
    if (random_num == 0) _self.payChannel.addTX( -1 )  
    return random_num
  }

  return {
    maketx    : maketx
  }
})
```

Run example https://dao.casino/vr (select pool)
