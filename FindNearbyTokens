/*
This script macro checks when tokens move in or out of range of a specified token ('targetToken'). 
If more tokens than 'numberOfTokens' come within range, it calls the 'enableEffect()' function. 
Conversely, if a token moves out of range resulting in there being less than 'numberOfTokens', it calls the 'disableEffect()' function.
These 2 functions will have to be configured for the specific purpose of this macro.

Filtering can be done by including or excluding friendly, neutral, hostile or hidden tokens.

Author: CDeenen
*/

////////////////////////////////////////////////////////////////////
//Configuration variables
////////////////////////////////////////////////////////////////////

let targetToken = 'selected';    //Set to 'selected' to get the token you've currently got selected, otherwise specify a token name
const maxDistance = 10;          //Max detection distance in ft
const numberOfTokens = 3;        //Minimum number of tokens to trigger the effect
const includeFriendly = false;   //Count friendly tokens
const includeNeutral = false;    //Count neutral tokens
const includeHostile = true;     //Count hostile tokens
const includeHidden = false;     //Count hidden tokens

const debug = false;             //Print debug info if set to true

////////////////////////////////////////////////////////////////////
//Functions that are run when an effect should be enabled/disabled
////////////////////////////////////////////////////////////////////

function enableEffect() {
    console.log("Enable effect!");
}
  
function disableEffect() {
    console.log("Disable effect!");
}

////////////////////////////////////////////////////////////////////
//No configuration below this point
////////////////////////////////////////////////////////////////////

//If target is set to 'selected', get the token name of the selected token
if (targetToken == 'selected') {
  if (canvas.tokens.controlled.length == 0) {
    ui.notifications.warn("No token selected");
    return;
  }
  if (canvas.tokens.controlled.length > 1) {
    ui.notifications.warn("Please select only one token");
    return;
  }
  targetToken = canvas.tokens.controlled[0].name;
}

//If token could not be found, return a warning
if (canvas.tokens.placeables.find(t => t.name == targetToken) == undefined) {
  ui.notifications.warn("Token could not be found");
  return;
}

//Check if the macro is already running
let alreadyRunning = game.findNearbyTokens != undefined;

//Register the settings
game.findNearbyTokens = {targetToken, maxDistance, numberOfTokens, includeFriendly, includeNeutral, includeHostile, includeHidden, debug}

//Set effectEnabled to false if this is the first time the macro is called
if (!alreadyRunning) {
  game.findNearbyTokens.effectEnabled = false;
}

if (game.findNearbyTokens.debug) console.log("Settings:",game.findNearbyTokens);

//Run 'checkNearbyTokens' function on macro start
checkNearbyTokens();

//If macro was already running, stop macro here
if (alreadyRunning) {
  ui.notifications.info("Macro reconfigured");
  return;
}

ui.notifications.info("Macro initialized");

//Whenever a token is updated, run 'checkNearbyTokens'
Hooks.on('updateToken', function(){
  checkNearbyTokens();
});
Hooks.on('deleteToken', function(){
  checkNearbyTokens();
});
Hooks.on('createToken', function(){
  checkNearbyTokens();
});

//This function checks all tokens and counts how many of them are close enough to the target token
function checkNearbyTokens() {

  //Stores the number of detected tokens within range
  let counter = 0;

  //Get the token object of the target token
  const token = canvas.tokens.placeables.find(t => t.name == game.findNearbyTokens.targetToken);
  if (game.findNearbyTokens.debug) {
    console.log("Checking for target token:", token.name, token);
    console.log("Origin of token:", token.x, token.y);
  }

  //Iterate all tokens on the canvas
  for (let t of canvas.tokens.placeables) { 

    //Exclude targetToken
    if (t.id == token.id) continue;

    //If the disposition should be ignored, continue to the next token
    if (!game.findNearbyTokens.includeHostile && t.document.disposition == -1) continue; 
    if (!game.findNearbyTokens.includeNeutral && t.document.disposition == 0) continue;
    if (!game.findNearbyTokens.includeFriendly && t.document.disposition == 1) continue;

    //If hidden tokens should be ignored, continue to the next token
    if (!game.findNearbyTokens.includeHidden && t.document.hidden) continue;
    
    //Measure the distance between the token and targetToken
    const distance = canvas.grid.measureDistance({x: token.x, y: token.y}, {x: t.x, y: t.y});

    //If the distance is greater than or equal to 'maxDistance', increment the counter
    if (distance <= game.findNearbyTokens.maxDistance) counter++;

    if (game.findNearbyTokens.debug) {
      console.log("Token to measure:",t.name,t);
      console.log("Origin:",t.x,t.y,"Distance:",distance);
    }
  }

  if (game.findNearbyTokens.debug) console.log("Number of tokens counted:",counter);

  //If the effect is not enabled and there are more tokens in range than 'numberOfTokens', run 'enableEffect'
  if(!game.findNearbyTokens.effectEnabled && counter >= game.findNearbyTokens.numberOfTokens) {
    enableEffect();
    game.findNearbyTokens.effectEnabled = true;
  }
  //Else if the effect is enabled but there are not enough tokens in range, run 'disableEffect'
  else if (game.findNearbyTokens.effectEnabled && counter < game.findNearbyTokens.numberOfTokens) {
    disableEffect();
    game.findNearbyTokens.effectEnabled = false;
  }
}