## Springboard 2024

Springboard is an 8-week program that I participated in. I was tasked with making a prototype of a game with a small team. 
The game we created is called Instrumentalia and it is a visual novel/ guitar hero-inspired rhythm game that can be played on itch.io <a href="https://ji117.itch.io/instrumentalia">here</a>. Also, you can view the code on the git repo <a href="https://github.com/ji117/Instrumentalia">here</a>. 
I was the project's lead programmer, implementing all major features. The other programmer credited with this project was mainly responsible for some additions to the UI and other minor changes/ adjustments.
The creation of this game was split into two parts. The visual novel side of the game and the rhythm game. The main challenge was the rhythm game as this was the first time I worked on a game of this nature. To help me with this I used FMOD, which was the first time I used the plugin as well, for its beat tracking capabilities and had plans to use its adaptive audio to change the song of the rhythm game based on player performance and player choices but this was outside the scope of the prototype.

Below is the game's trailer (note the trailer was not created by me) 

{% include youtube.html id="GNDAZwnszO4" %}


Springboard also allowed me to do a bit of freelance work since I got the chance to help other teams on the program with their games. 
One of these games was called Data Breach which you can see <a href="https://teamc.itch.io/data-breach">here</a>. I made some minor adjustments to the enemy AI where to move away from the player when spotted as they wanted it to try and sneak up to the player before attacking and made it so that the enemy could damage the player. I also implemented the logic for the win/lose condition of the game. 
The other game that I worked on was called Acorn Rumble which can be seen <a href="https://jasonts.itch.io/acorn-rumble">here</a>. I helped out with sorting out the player controller since there were issues with getting it to behave as intended which I did by changing it from using Unity's old input system to the new input system. I added a timer for the turn system. Added a screen shake feature and made a way for the screen shake to be consistent between the four cameras used in the game. I also added logic of when to call the functions related to the game's cheating mechanics. Added a player-select screen and helped with the identification and fixing of bugs. 

Below are code snippets from the game my team made.

Coroutine to type dialogue out letter by letter 
```
        dialogue.text = "";
        state = State.PLAYING;
        int wordIndex = 0;

        typingEventEmitter.Play();
        typingEventEmitter.EventInstance.setVolume(volume);
        while(state != State.COMPLETED)
        {
            if (typingEventEmitter.IsPlaying() == false)
            {
                typingEventEmitter.Play();
                typingEventEmitter.EventInstance.setVolume(volume);
            }
            
            dialogue.text += text[wordIndex];
            yield return new WaitForSeconds(dialogueSpeed);
            if(++wordIndex == text.Length)
            {
                state = State.COMPLETED;
                break;
            }
        }
        typingEventEmitter.Stop();
```
Function to play next sentence and change name and sprites to the character currently speaking
```
        currentDialogueLine = currentScene.sentences[++sentenceIndex].text.Replace(">PLAYER NAME<", playerName);
        StartCoroutine(TypeDialogue(currentDialogueLine));
        speakerName.text = currentScene.sentences[sentenceIndex].speaker.speakerName;
        speakerPotrait.sprite = currentScene.sentences[sentenceIndex].speaker.speakerPotrait;

        if (currentScene.sentences[sentenceIndex].speaker.speakerInstrument != null)
        {
            instrument.gameObject.SetActive(true);
            instrument.sprite = currentScene.sentences[sentenceIndex].speaker.speakerInstrument;
        }
        else
            instrument.gameObject.SetActive(false);

        speechEventEmitter.Play();
        speechEventEmitter.EventInstance.setVolume(volume);
```
Function to spawn notes for Rhythm game. This is called each new bar in the song 
```
            if (nextSpawn > 0)
            {
                nextSpawn--;
            }
            else
            {
                int min = 1;
                int max = 5;
                int spawner = Random.Range(min , max);
                
                switch(spawner)
                {
                    case 1:
                        spawnerToUse = spawner1;
                        break;

                    case 2:
                        spawnerToUse = spawner2;
                        break;

                    case 3:
                        spawnerToUse = spawner3;
                        break;

                    case 4:
                        spawnerToUse = spawner4;
                        break;
                }
                
                var obj = Instantiate(objectToSpawn, spawnerToUse.transform);
                nextSpawn += spawnInterval;
                obj.transform.position = spawnerToUse.transform.localPosition;
            }
```
Code to tell the difference between a good and a perfect hit when a note is hit
```
            if (Input.GetKeyDown(key) && detector.IsBothColliding() && delay <= 0)
            {
                noteCollidedWith.gameObject.SetActive(false);
                Destroy(noteCollidedWith.gameObject);
                GameController.gameInstance.AddScore(200);
                GameController.gameInstance.AddPerfect();
            }
            else if (Input.GetKeyDown(key) && isNoteColliding && delay <= 0)
            {
                noteCollidedWith.gameObject.SetActive(false);
                Destroy(noteCollidedWith.gameObject);
                GameController.gameInstance.AddScore(100);
                GameController.gameInstance.AddGood();
            }
```
