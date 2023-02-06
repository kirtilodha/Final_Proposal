# Google Summer of Code 2022 Proposal

# Personal Details
**Name:** Kirti Lodha

**Email:** kirtilodha0@gmail.com

**College Name:** Dayananda Sagar College of Engineering 

**Degree:** BE 

**Expected Graduation Year:** 2023

**LinkedIn:** [LinkedIn](https://www.linkedin.com/in/kirti-lodha-b05059199/) 

**GitHub:** [Github](https://github.com/kirtilodha) 

**Resume -** [Resume](https://drive.google.com/file/d/17rcowIejO_kCBAZaii-6kKbJHgiQIXmW/view?usp=sharing)

**Primary Spoken Language:** English

# Chosen Idea: Agora Blockchain
## Abstract
The basic features in an Election DApp are user registration, election creation, voting and result calculation. To make our DApp more flexible and user friendly, we need to add more voting algorithms apart from General Voting Algorithm. The three important features that need to be implemented are 1) Open and Invite based elections. 2) Authentication and user KYC. 3) Multiple algorithms to conduct elections.

# I. Technical Details
This section is divided into 4 parts that include all three features needed and some additional features that might look good to our DApp.
## 1. Multiple algorithms to conduct elections.

There are multiple voting algorithms, each of which has its advantages and disadvantages. The possible algorithms were found [here](https://codein.withgoogle.com/archive/2018/task/5392965283348480/). All the algorithms can be displayed as a card to let the user know about the algorithm.
<br>

### 1.1 Range Voting
In this voting system, each voter can rate each candidate in a range of 1 to 5. In the simplest form, the scores of each candidate from multiple voters are summed and the candidate with the highest score wins the election.

![](https://i.imgur.com/wdg0LxZ.png)

#### 1.1.1 Rationale behind approach
* The system will be independent i.e., removing even one candidate won't disturb the ratings.
* This voting algorithm allows the voter to give the same rank to different candidates. It respects the user's choice if he/she wants to give the same rating.

#### 1.1.2 Implementation Details

**Front-end:** On the front-end side, the manager will be given several choices to choose one voting algorithm. This is done using the radio inputs button to ensure only one algorithm is chosen at a time.

```
Which type of Election you want?
<div>
    <input type="radio" value="General" name="algo" onChange={handleAlgo}/> General Voting
    <input type="radio" value="Range" name="algo" onChange={handleAlgo} /> Range Voting
    <input type="radio" value="Others"name="algo" onChange={handleAlgo} /> Others
</div>
```

**Back-end:** The input is handled using states in Reactjs. The code below sets the value of the algorithm as General/ Range/ Others, depending on what the user has chosen.

```
const handleAlgo = (e) => {
    setNda({
        ...nda,
        algorithm: e.target.value
    });
}
```

The algorithm is then passed as an argument to create the election which now allows one to vote using the specified algorithm. Two states have been created to keep a track of the candidate id and the value given:
```
const [id,setId]=useState(null);
const [val,setValue] = useState(0);
```
The front-end changes according to the input using the following code:
```
<label className="voteCandidate">
    {currentElectionDetails?.info?.algorithm == "General"?
    <input type="radio" name="candidate" value={candidate?.id} onChange={handleCandidateIdChange} className="voteCandiateInput" />:
    <></>}
    
    <Candidate name={candidate?.name} id={candidate?.id} about={candidate?.about} voteCount={candidate?.voteCount} imageUrl={AVATARS[candidate?.id % AVATARS?.length] || '/assets/avatar.png'} />
    { currentElectionDetails?.info?.algorithm == "Range" ?
    <>
    <ReactStars
    className="voteCandiateInput"
    count={5}
    onChange={(newValue) => {
        setValue(newValue);
        setId(candidate?.id);
        }}
    size={24}
    color2={'#ffd700'}
    value={0}
    half={false} />
    <Button ml={3} type="submit" onClick={handleVoteSubmit}>Confirm</Button>
        </>:
    <p></p> 
    }
</label>
```

When the voter votes using the specified algorithm, different functions need to be invoked in `VoteModal.js` to handle the different parameters involved. In this case, if the algorithm is chosen as Range, then the `vote_range` function is called.
```
if(currentElectionDetails?.info?.algorithm == "Range"){
    await CurrentElection.vote_range(id,val).send({ from: account });
}
else if(currentElectionDetails?.info?.algorithm == "General"){
    await CurrentElection.vote_general(candidateId).send({ from: account });
}
```
The required functions for different algorithms can be written in the smart contract. Since solidity does not support switch statements, we can use if statements to check the rating given. A mapping can be useful to keep a track of the candidate that a voter has already voted for. This will be helpful to ensure the voter does not vote again.
```
mapping(address=>uint[]) voting_to_id;
```
The  Solidity code showing the algorithm for the range voting algorithm is shown below: 
```
function vote_range(uint _candidate,uint val) public {
    bool flag=false;
    for(uint i=0;i< voting_to_id[msg.sender].length;i++){
        if(_candidate == voting_to_id[msg.sender][i]){
            flag=true;
        }
    }
    require(!flag, "Voter has already Voted!");
    require(_candidate < candidatesCount && _candidate >= 0, "Invalid candidate to Vote!");
    require(getStatus() != Status.closed, "Election closed");
    require(getStatus() != Status.pending, "Election not yet started");
    voting_to_id[msg.sender].push(_candidate);
    if(val==5){
        candidates[_candidate].voteCount=candidates[_candidate].voteCount+5;
    }
    else if(val==4){
        candidates[_candidate].voteCount=candidates[_candidate].voteCount+4;
    }
    else if(val==3){
        candidates[_candidate].voteCount=candidates[_candidate].voteCount+3;
    }
    else if(val==2){
        candidates[_candidate].voteCount=candidates[_candidate].voteCount+2;
    }
    else if(val==1){
        candidates[_candidate].voteCount=candidates[_candidate].voteCount+1;
    }
    electionInfo.voterCount++;
}
```
The algorithm is as follows:
* Take two parameters: one who has been voted and the second what rank it received.
* We need to check if the voter has already voted for the candidate. This is done using a loop to check if the candidate id is already present in the array. If yes, then the voter has already voted, otherwise not.
* We also need to check the election's status and the voter is allowed to vote only when it is active.
* If all the checks are passed, we push the candidate id into the array and check for the rating value.
* According to the value, the count gets added up to the candidate's voteCount.

At last, the results for the Range Voting are determined by the same process as the General Voting System.
#### 1.1.3 Demonstration video: [https://youtu.be/pBfxEnaYEfw](https://youtu.be/pBfxEnaYEfw)
### 1.2 Borda Voting Algorithm
The Borda algorithm is very similar to Range Voting Algorithm except for the fact that the Borda algorithm does not allow the same rating to be given to multiple candidates. In this case, the number of ratings will be equal to the number of candidates.
#### 1.2.1 Implementation Details
The Borda algorithm requires an additional check to ensure that the rating is unique for every candidate. This can be achieved using an array that can store the ratings. Any rating input must be checked using a required statement at first.

### 1.3 Kemeny–Young Voting Algorithm

The Kemeny–Young method uses pairwise comparison counts and preferential ballots to determine which candidates are most popular.
This method can be divided into two steps:
* Voters will be voting for all possible number of pairs and then a tally table can be used to summarise the total number of votes preferred.
* The second step is to calculate the score for each pair, compare their scores and test all possible rankings.
Let's take an example where we have 3 candidates - a, b, c:


| All possible pairs | Number of votes    | Number of votes    | Number of votes  |
| ------------------ | ------------------ | ------------------ | ---------------- |
|                    |  Preferred X over Y   |Preferred Y over X     | Equal preference  |
|  X=a , Y=b    | 10                   |40                    |    0              | 
| X=a , Y=c                    |30                   |     10              | 10                |
|  X=b , Y=c                   |25  | 20 | 5 |

In this example, for the first case when X=a, Y=b, 10 votes are in favour of a, 40 votes are in favour of b and 0 votes are in favour of both. 

### Calculating the overall ranking
In the case of a the total number of votes preferred are 10+30+10=50 whereas in the case of b, the total number of votes preferred is 40+25+5=70, and in the case of c, the total number of votes preferred is 10+10+20+5=45. In this case, the ranking will be `b>a>c`. 
#### 1.3.1 Implementation Details
* We can use two loops in the Candidates array to display all possible pairs. 
* Voters will be allowed to vote for each pair according to their preference and initiate a transaction.
* Suppose in case of X=a, Y=b, the voter votes for a. Then the voteCount for a will increase by 1. Similarly, for all other candidates, the voteCount can be stored.
* At the end, these voteCounts can be compared and ranked from highest count to lowest.

**Why these methods?** <br>
According to Wikipedia and after doing research it has been seen that the above-mentioned algorithms have additional benefits over other algorithms. This image has been taken from the [wikipedia](https://en.wikipedia.org/wiki/Schulze_method).
![](https://i.imgur.com/cDyMSbH.png)

**If time permits, I would like to implement more algorithms into our DApp. The next one would be `Schulze`.** Schulze method also involves pair-wise comparison and a Condorcet winner that is the winner is preferred over every other candidate.
## 2. Authentication and user KYC

Elections are very critical for an organization, and anyone with a wallet and few tokens in it should not be allowed to vote until they are KYC verified. But to increase the user engagement, we can give the organiser the right to choose if he wants the voters to be KYC verified or not. 

**Why KYC verification should not be mandatory?**

For example, when there is an art contest on Instagram, the one who gets the maximum likes on their post wins. In that case, there's no restriction to get people from all over the world to like the post. Similarly, if the election is held through our DApp, people can circulate the election link to get as many people on board to vote for them. This will let more people know about our DApp.

#### 2.1 Implementation Details for KYC verification
We need to capture customer information before onboarding them for voting. This process will require 3 main steps:
* Upload documents for verification
* Admins to verify the documents
* User getting verified to vote

The best way to upload documents in a secured manner is by using IPFS. The InterPlanetary File System (IPFS) is a peer to peer file- sharing protocol connecting computing devices for sharing/ storing files. The below image shows a glimpse of what IPFS does.

![](https://i.imgur.com/qcjSCtv.png)


We can use the IPFS network to upload and retrieve KYC docs ensuring an end to end encryption. Before sharing out the KYC docs into the IPFS network, we consider encrypting the file for extra security and reducing the file size, since anyone accesses the KYC docs from the IPFS network by just knowing their hash values.
I'm proposing this idea from [this](https://ieeexplore.ieee.org/document/9230987) research paper. 

Three different approaches can suit well for our Election DApp.

#### 1. When there's a single admin
Digital signatures are used to prove the documents without exposing their private keys. In our application, when the user uploads the document, an encryption function is run in the backend which takes the public key of the admin and the document and encrypts it. While on the admin side, during verification, the encrypted hash can be decrypted using the admin's private key and hash generated. The admin can then access the document and verifies it. I'll be referring to this detailed [article](https://medium.com/mycrypto/the-magic-of-digital-signatures-on-ethereum-98fe184dc9c7) on the medium website.

But there is huge human dependency in the above-mentioned process. The following flow diagram aims to reduce the human dependency.
![](https://i.imgur.com/qtcQyz5.png)

To reduce that, we can apply a Machine Learning algorithm at first which uses image recognition and compares the face capture and the document given. 

#### 2. When there are multiple admins
In this case, whenever a user requests KYC verification and uploads their document, this document can be stored on the IPFS network as discussed above. The idea to have multiple admins is to create a decentralised environment in case one admin is bribed for conducting the elections. The admins will have an option to verify or reject the KYC request. When the user gets approved by 50% of the admins, he/she gets KYC verified and the right to vote in the election. 
#### 3. If voter ids are available on a public database
In the current scenario, there's an API through which we can check if an Aadhar card is valid or not. If we find a similar kind of trusted database we only need to check their voter id along with their public address to see if they are valid or not.

Ideas have also been taken from [this](https://github.com/amand1996/Votechain).
The verified voters will be stored on-chain and they can show their verification details to third-party also.
## 3. Open and Invite based elections
The idea is to provide two types of election: open and invite-based. Open elections are elections in which anyone can vote provided that they are KYC verified and invite based elections allow the organisers to send the voters email invites with a secure single-use link. Using that link a voter can vote without KYC verification. These voters will be added to the blockchain, let's say to an array `invited`.
<br>
#### 3.1 Implementation Details
While creating an election, the organiser can choose the type of election he/she wants. If he/she chooses the invite-based election, then a second box is popped up to enter the email and submit for sending the election link to the voter's email id. The election link can be captured from the back-end. Each invite will cost some gas price to the organiser.
![](https://i.imgur.com/TeJ9V6M.png)

We can use [MailJS](https://www.emailjs.com/), and use [SendGrid](https://github.com/sendgrid/sendgrid-nodejs) for transactional services. From the front end side, we can send a request to an SMTP server, which will send the email.

A check can be put to see if the election is open or invite-based. If it is open, then the voters have to be KYC verified and in case of an invite-based election, we remove the KYC verification check to allow them to vote without being verified. Before voting, there will be a check to ensure that the voter is present in the `invited` array.

## 4. Extra features

* **Sign up as `abc.voter` and earn an NFT** - While signing up, `abc.voter` will be a domain we would provide to get them identified as a voter for using our platform. If I were a voter, I would get `kirti.voter`. 

    **Why do we need this?**
    * Since we have elections in which KYC is not mandatory, anyone can vote, at least some identity should be there as to who's voting and not any bots doing this. 
    * This will act as a sort of minimal verification to sign up on our platform.
    * The concept of NFTs can be used to increase user engagement and self-identity.

So the user enters the election dashboard with its identification to see all the elections going on. If an election needs KYC, before voting, the user will be prompted to verify themselves.

**Approach-**

I've implemented the Voting Name Service which can be seen [here](https://voting-name-service.vercel.app/).
It takes the name of the user and creates a domain `abc.voter`  and mints an NFT on the same name. This NFT is accessible and viewable on OpenSea. Few NFTs which I created can be seen [here](https://testnets.opensea.io/collection/voting-name-service).

![](https://i.imgur.com/nDD5wgY.png)


* **Restrict Dates** - In the present scenario, an election can be created at any date(even if passed), which will ultimately does not allow the organiser to add candidates. An election should only be created after the current date and time. 
```
<DatePicker
    required
    showTimeSelect
    dateFormat="yyyy/MM/dd hh:mm:ss"
    className="form-control"
    name="endTime"
    selected={se.endTime * 1000}
    onChange={(date) => handleSeChange(date, "end")}
    minDate={new Date()}
/>
```
The line `minDate={new Date()}` will restrict the past dates to get selected. This can be implemented using CSS properties.
* **Extend Election dates** - The organiser might want to extend the election dates. There must be an option to do so. This will cost some gas price because it will require a change in the blockchain. Accordingly, a function in Solidity and states in Reactjs would easily do the job. We can take the new extended date input and update the state and call the `onChange` function which in return will invoke a function in our smart contract to update the variable `uint edate`.
* **Required fields** - In the current application, an election can be created without providing the fields like `Create Election`, and `Add Candidate`. These fields should be made mandatory to give the users clarity as to whom they're voting for.
* **Settings button** -
    * Logout - As of now, we cannot disconnect Metamask using Ethereum API. Reference - [here](https://stackoverflow.com/questions/67856902/how-to-logout-of-metamask-account-using-web3-js)
    So, if the user clicks the logout button, then we can show them the procedure to disconnect the wallet.
    * KYC verification - From this button, the user will be able to start its KYC verification.
    * Profile edit - A function in Solidity can be written to update the name of the user.
* **Delete button** - There has to be a button through which an organiser can delete an election if they want. Since data cannot be altered once in Blockchain, we can remove the election from our UI. This can be done by keeping a track of deleted elections, and while rendering the current elections, check if the election is marked as deleted. If yes, then we don't render it.
* **Improve the UI** - The below-mentioned image shows that the hovering text is not visible. This can be corrected through CSS.
![](https://i.imgur.com/JxbwJhV.png)
* **Shorten the visibility of contract address** - The full contract address can make a user uncomfortable during voting. We can show a few first and last digits, like `0xaD37....D081`. This can be done using the following code:
```
Wallet: {infoText.slice(0, 6)}...{infoText.slice(-4)}
```
* **Switch Network Functionality** - Currently, if the user is not connected to the `Avalanche Fuji Chain`, the public address does not show up and there's no way the user would understand that he might be in the wrong network. 
# II. Schedule of Deliverables
<br><br><br>
![](https://i.imgur.com/VElT494.jpg)

<br><br>

### 0. Before Community Bonding

* Get familiar with the Agora Blockchain codebase.
* Be ready with a plan and ideas to discuss

### 1. Community Bonding Period
#### 1.1 Week 1:
* Get in touch with my project mentor, introduce myself in the community, and work out a good time to communicate (taking into account timezone differences and their schedule).
* Ask the mentor for his ideas/plans for the project. The project proposal reflects my knowledge of writing, there may be essential amendments required and it would be better to make them early.
#### 1.2 Week 2:
* Go through the Agora Blockchain codebase thoroughly. Make notes of the functions/files that might be helpful in future.
* Explore the existing solutions and keep a track of the best approaches found.
* Get in contact with other GSoCer's , both within the AOSSIE community and outside.
#### 1.3 Week 3:
* Discuss the current approach with the mentor and start noting down their advantages and disadvantages of them.
* Come up with the possible approaches and finalize what all features are required.
* Start laying out a general outline of the possible cases.
### 2. Phase 1
#### 2.1 Week 4:
* Start with the algorithms that were finalised.
* Note the essential attributes, functions, and data structures required.
* Layout a basic outline explaining the algorithm.
* Update the UI.
* Write functions in Solidity to implement the logic behind each voting algorithm.
* Unit testing of every function needs to be done.
#### 2.2 Week 5:
* Integrate the front-end with the smart contract.
* Create separate smart contracts to avoid dependencies and they should be loosely coupled.
* Test and finalise the algorithms and get feedback from the mentor.
* Implement the feedback received from the mentor.
#### 2.3 Week 6:
* Complete any left-over work.
* Start planning on KYC verification and its implementation.
* Look for resources and go through certain research papers.
#### 2.4 Week 7:
* Start reading on IPFS and start implementing its basics.
* Integrate the document upload feature.
* Read on Digital signatures, encryption and decryption processes.
#### 2.5 Week 8:
* Write functions to encrypt and decrypt data using the uploaded document.
* Integrate the encryption, decryption, and verification processes.
* Give the mentors an update on my progress so far and take their feedback.
#### 2.6 Week 9:
* Finalise the KYC verification
* Update the UI and rest scripts to use verification first.
* Handle any error and debugging period.

### 3. Phase 2
#### 3.1 Week 10:
* Elections can be open or invite-based elections.
* Integrate this feature using EmailJS and SendGrid.
* Implement the front-end part.
#### 3.2 Week 11:
* Testing and compiling phase.
* Handle any unexpected errors.
* Deploying on the test net.
* Complete any left-over work.
#### 3.3 Week 12:
* Work on the extra features like the change of CSS properties, and improve the UI.
* Implement the settings button.
* Organisers must be allowed to extend the election dates.
#### 3.4 Week 13:
* Testing each function and asserting it to check errors.
* Checking the application with different test cases.
* Finalize and complete the documentation.
* Refactor Code.
#### 3.5 Week 14:
* Deploying on the Avalanche Fuji Chain
* Checking the entire application thoroughly
* Work on leftover stuff.
#### 3.6 Week 15:
* Buffer week for unexpected delays, final submissions for GSoC 2022.


# III. Skills
### Programming Skills
>* Solidity
>* Fluent in C++, Python
>* Good knowledge of HTML, CSS, JS, Reactjs
### Blockchain Skills
>* Good knowledge of Blockchain basics including Solidity, Metamask, Web3, etc.
>* Completed a Blockchain specialisation course by Buffalo University
>* Involved in finding various Blockchain applications and knowing them better
>*  Comfortable with Git and GitHub
# IV. Previous Blockchain Project Development Experience
* CrowdFunder - [https://github.com/kirtilodha/CrowdFunder](https://github.com/kirtilodha/CrowdFunder)

A decentralized platform that allows developers all over the globe to gather funding for their projects. They can request a campaign and provide details about their project. To maintain transparency, developers can spend money only if 50% of the contributors approve.

**Implementation video**: [Link](https://github.com/kirtilodha/CrowdFunder/blob/master/README.md)

* Voting Name Service - [https://github.com/kirtilodha/Voting-Name-Service](https://github.com/kirtilodha/Voting-Name-Service)

This is part of an Election Dapp which uses a ".voter" to signup for the DApp. A voter can earn an NFT while registering and minting a domain for their name.

**Implementation**: [http://voting-name-service.vercel.app/](http://voting-name-service.vercel.app/)

* MintNFTs - [https://github.com/kirtilodha/MintNFTs](https://github.com/kirtilodha/MintNFTs)

A platform to mint NFTs and these are accessible in OpenSea.

**Implementation**: [https://mint-nfts-cyan.vercel.app/](https://mint-nfts-cyan.vercel.app/)
<br>
# V. Other Open-Source development experience
* **Kharagpur Winter of Code** - [https://github.com/kirtilodha/ElectionDapp](https://github.com/kirtilodha/ElectionDapp)
    * The project was based on an Election DApp.
    * I contributed to the front-end as well as back-end side.
    
    **Merged PR**:
[https://github.com/Shikhar15606/ElectionDapp/pull/136](https://github.com/Shikhar15606/ElectionDapp/pull/136)
* **Hacktoberfest** - Mattermost project - [https://github.com/kirtilodha/focalboard](https://github.com/kirtilodha/focalboard)
    * Created issues 
    * Enhanced their webpage
    * Resolved 3 issues regarding interaction with the webpage
    
    **Merged PR**
    [https://github.com/mattermost/focalboard/pull/1669](https://github.com/mattermost/focalboard/pull/1669)
    
    **PRs**
    *   [https://github.com/mattermost/focalboard/pull/1650](https://github.com/mattermost/focalboard/pull/1650)
    *    [https://github.com/mattermost/focalboard/pull/1649](https://github.com/mattermost/focalboard/pull/1649)
    
    
* **GsSOC** – Jagrati WebApp - [https://github.com/kirtilodha/JagratiWebApp](https://github.com/kirtilodha/JagratiWebApp)
    * Created issues
    * Resolved the existing issues
# VI. Why this Project?
The project is based on Election DApp upon which I've previously worked. I'm very familiar with the tech stack it uses - Reactjs, Solidity, Ganache, Metamask, Web3. When I looked around the codebase and was integrating the Range Voting Algorithm, I was sure that I can integrate the required features and bring a lot more interesting features onto the table. 
* **Different algorithms:** I found this feature very fascinating and informative. When I implemented the Range Voting algorithm, it created more curiosity to implement the other algorithms.
* **Flexibility and User experience:** Due to its open and invite based feature, the DApp can be used for several different types of purposes.
* **A chance to be a part of an open-source community:** I want to take up this opportunity to work in a large scale open-source organisation.
# VII. Do you have any commitments during the GSoC Period?
I have no commitments for this summer and will be available to contribute to the project.
<br> <br>

# VIII. How many hours will you work per week?
I'll work for 15+ hours per week.
# IX. About Me
I'm Kirti Lodha currently in 3rd year pursuing Electrical and Electronics engineering from 
DSCE, Bangalore. I'm currently working as a GitHub Extern'22, in which the project is based on Metaverse. I also have been selected as an MLSA in April 2020. I was among the top 12 
selected in WIEHACK 3.0 hackathon. Hackathons, innovations, projects, and webinars are my 
favourite kinds of work. 
<br>
I’m a Blockchain enthusiast and a competitive programmer. I code in C++ and have good 
knowledge on HTML, CSS, JS, Reactjs. I’ve been coding in Solidity for a year now, which is based on C++ 
and JS. Earlier I worked on research papers with a professor from my university, DSCE.
# X. Blogs
* [Operators in Python](https://www.scaler.com/topics/python/operators-in-python/)
* [range-function-in-python](https://www.scaler.com/topics/range-function-in-python/)
* [Leap year program in C](https://www.scaler.com/topics/leap-year-program-in-c/)
# XI. Why Me?
I've extensively used Solidity language and Reactjs to make personal projects and explored the field of data privacy, secured transactions, and contract creation. I've previously worked on projects which had similar concepts and contributed to various open-source organisations. I have contributed to Agora Blockchain for around a month now which has helped me to get familiar with the source code and the project requirements.
I've maintained proper documentation for every project for easier understanding in the upcoming times.

I'm fairly confident to work on the project and make it a success.


---

### Thank You






