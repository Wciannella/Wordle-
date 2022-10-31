# Wordle-
Here is a wordle dupe code written in python. Written with for and while loops, recursion, comprehension and iteration. 

# Import two useful items.
from random import choice
from string import printable

######################################################################
# Specification: readWords(filename) takes a single value, a string,
# the name of a file that contains a list of 5-letter words (one per
# line) consisting of the university of 5-letter words for the Wordle
# puzzle. It returns a list of words.
#
def readWords(filename='words (1).dat'):
    with open(filename, 'r') as infile:
        S = infile.read().lower().split() # Words
    #print("{} words read.".format(len(S)))
    return(S)

######################################################################
# Specification: evalGuess(guess, target) takes two equi-length
# lower-case strings representing the user's latest guess and the
# hidden target word, respectively. It returns feedback string that is
# equal in length to both word and target, consisting of either upper
# or lower case characters from word or the '.' character.
#
def evalGuess(guess, target):
    fback = []   # Construct feedback
    pool = []    # Remaining unmatched target letters

    # Find matches between guess and target while building pool of
    # unmatched letters from target.
    for i in range(5):
        if guess[i] == target[i]:
            # Match; update fback.
            fback.append(guess[i].upper())
        else:
            fback.append('.')
            # Add unmatched target letter to pool.
            pool.append(target[i])

    # Rescan looking for misplaced letters that are still left in the
    # pool; replace with lower cased letter in fback while ensuring no
    # spurious duplication of partial matches by "consuming" the pool
    # letter.
    for i in range(5):
        if guess[i] != target[i] and guess[i] in pool:
            # Match, but not in that position.
            fback[i] = guess[i]
            pool.pop(pool.index(guess[i]))

    # Splice the feedback back together and return it.
    return(''.join(fback))

######################################################################
# consistent(feedback, word) returns True if the feedback is
# consistent with word. To be consistent, word must contain all of the
# upper case letters in feedback, and, once these are accounted for,
# must contain all of the lower case letters of feedback once the
# letters corresponding to upper case feedback letters are excluded.
#
# This is a tricky function.
def consistent(feedback, word):
      list = [True,True,True,True,True]
      for i in feedback:
            if i.islower() and i not in word:
                  list[0] = False
      if feedback[0].isupper() and feedback[0].lower in word[0]:
            list[0] = False
            pass
      if feedback[1].isupper() and feedback[1].lower in word[1]:
            list[1] = False
            pass
      if feedback[2].isupper() and feedback[2].lower in word[2]:
            list[2] = False
            pass
      if feedback[3].isupper() and feedback[3].lower in word[3]:
            list[3] = False
            pass
      if feedback[4].isupper() and feedback[4].lower in word[4]:
            list[4] = False
            pass
      return (all(list))

######################################################################
# calcSize(S, feedback) returns the number of words in S that are
# consistent with the feedback provided.
#
def calcSize(S, feedback):
    AW = 0
    for word in S:
        if (consistent(feedback,word)):
            AW+=1
    return AW 

######################################################################
# Finally, we're ready to implement the wordle() game management
# function. Here, I will provide a framework for you to
# complete. Search for the comments containing "FIXME" for more
# detailed instructions.  But first, we'll need to import the choice()
# funcion from the random module to select a target word from S at the
# start of the game.
from random import choice

# Specification: wordle(S) takes a list of words, S, and randomly
# selects a target word for this round from S. It then manages game
# play, and returns True (meaning the user would like to play another
# round) or False (meaning that the user does not wish to play another
# round).
#
# Game management means (i) printing a suitable prompt, (ii) taking
# the user's input, (iii) rejecting the input if it is not "legal,"
# (iv) responding to a special command (here, '.', '+', or '?'), or
# (v) evaluating the input against the target word (see previous
# function). The process repeats until (i) the user guesses the target
# word, (ii) the makes six guesses without guessing the target word,
# or (iii) the user chooses to abruptly end the game via a special
# command.
#
def wordle(S):
    target=choice(S)			# Random target word
    osize = len(S)			# Initial size of dictionary
    feedback = '.'*5			# Initial feedback is empty
    avail = list(printable[10:36])	# List of available letters
    history = ""			# String history of word + feedback
    n = 6				# Remaining guesses

    # Print opening banner
    print('Welcome to Wordle!')
    print('Enter your guess, or ? for history; + for new game; or . to exit')

    # Repeat while guesses remain (or user quits with '+' or '.'
    # input). The general idea of this main game loop is to prompt the
    # user for a guess, then process the guess accordingly.
    while n > 0: 
        # Uncomment the following line to enable "cheat mode." Cheat
        # mode reveals the usually hidden target word, making it
        # easier for you to debug your code.
        print("Cheat: {}".format(target))  # Cheat mode!

        # Show available letters.
        print ('Available Letters:',(''.join(avail)))
        pass

        # Prompt the user for a guess. 
        guess = input("\nWordle[{}]: '{}' >  ".format(7-n, feedback))

        # The next conditional statement breaks down game management
        # into an appropriately ordered series of outcomes.
        if guess in '.+':           # Abort game
            print("Abort game.\n{}The target word was {}\n".format(history, target))
            return(guess=='+')
        elif guess=='?':	    # Show guess history
            print("History:\n{}".format(history))
            continue
        elif guess in history:
            print('You have already guessed that!')
            continue
        elif guess not in S:        # Illegal guess
            print("Unrecognized word: '{}'".format(guess))
            continue
        elif not consistent(feedback, guess):
            print("Successive guesses must use all {} known hints.".format(5-feedback.count('.')))
            continue
        feedback = evalGuess(guess, target)
        # Guess is legal and fits known feedback; accept the guess and
        # check it for the win.
        history = history + " {}: {} => {}\n".format(7-n, guess, feedback)
        if feedback.lower() == target: # You win!
            print("Good job!\n{}The target word was {}\n".format(history, target))
            return(True)

        # Not a winner (yet): update status and try again. 
        nsize = calcSize(S, feedback)	# new size
        print("Quality: {:.2%} [{} words remain]".format((osize - nsize + 1)/osize, nsize))
        osize = nsize

        # Update list of available letters. Unlike the real game, we
        # can't shade letters, so the semantics here will be a bit
        # different: an upper case letter is in the word, while lower
        # case are as yet undetermined.
        char= []
        notchar= []
        for i in guess:
            if i in list(target):
                char.append(i)
            if i not in list(target):
                notchar.append(i)
        for L in range(len(avail)):
            if avail[L] in char:
                avail[L] = avail[L].upper()
            if avail[L] in notchar:
                avail[L] = '.'

        # Go on to next guess
        n = n-1

    # If you get to this point, the user has run out of
    # guesses.
    print("Sorry, no dice...\n{}The target word was {}\n".format(target, history))
    # Assume they want to play another game.
    return(True)

######################################################################
# This code is executed automatically when you feed this file to a
# fresh invocation of Python. So, for example, from the Linux prompt:
#    > python hw1.py
# will start the game for interactive use right at the shell.
#
if __name__ == '__main__':
    # Read in the list of legal 5-letter words and then continue
    # playing the game until wordle() returns False.
    while wordle(S=readWords('words.dat')):
        print("Let's play again!\n")
