# Google Summer of Code 2022 Proposal

---

# Personal Details
**Name:** Kirti Lodha

**Email:** kirtilodha0@gmail.com

**College Name:** Dayananda Sagar College of Engineering 

**Degree:** BE 

**Expected Graduation Year:** 2023 

**LinkedIn:** [LinkedIn](https://www.linkedin.com/in/kirti-lodha-b05059199/) 

**GitHub**: [Github](https://github.com/kirtilodha) 

**Mobile:** +91 9331320052 

**Resume** - [Resume](https://drive.google.com/file/d/17rcowIejO_kCBAZaii-6kKbJHgiQIXmW/view?usp=sharing)

**Primary Spoken Language:** English

# Chosen Idea: Agora Blockchain
### Abstract
The basic features in an Election DApp are user registration, election creation, voting and result calculation. In order to make our DApp more flexible and user friendly, we need to add more voting algorithms apart from General Voting Algorithm. The three important features that needs to be implemented are 1) Open and Invite based elections. 2) Authentication and user KYC. 3) Multiple algorithms to conduct elections.

# I. Technical Details
This section is divided into 4 parts that includes all three features needed and some additional features that might look good to our DApp.
## 1. Multiple algorithms to conduct elections.

There are multiple voting algorithms, each of which has their own advantages and disadvantages. The possible algorithms were found [here](https://codein.withgoogle.com/archive/2018/task/5392965283348480/).
### 1.1 Range Voting
In this voting system, each voter can rate each candidate at a range of 1 to 5. In the simplest form, the scores of each candidate are summed and the candidate with the highest score wins the election.

![](https://i.imgur.com/wdg0LxZ.png)

#### 1.1.1 Rationale behind approach
* The system will be independent i.e., removing even one candidate won't disturb the ratings.
* This voting algorithm allows the voter to give same rank to different candidates. It respects the user's choice if he/she wants to give the same rating.

#### 1.1.2 Implementation Details

**Front-end:** On the front-end side, the manager will be given a number of choices to choose one voting algorithm. This is done using radio input button to ensure only one algorithm is chosen at a time.

```
Which type of Election you want?
<div>
    <input type="radio" value="General" name="algo" onChange={handleAlgo}/> General Voting
    <input type="radio" value="Range" name="algo" onChange={handleAlgo} /> Range Voting
    <input type="radio" value="Others"name="algo" onChange={handleAlgo} /> Others
</div>
```

**Back-end:** The input is handled using states in Reactjs. The code below sets the value of algorithm as General/ Range/ Others, depending on what the user has chosen.

```
const handleAlgo = (e) => {
    setNda({
        ...nda,
        algorithm: e.target.value
    });
}
```

The algorithm is then passed as an argument to create the election which now allows to vote using the specified algorithm. Two states have been created to keep a track of the candidate id and the value given:
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

When the voter votes using the specified algorithm, different functions needs to be invoked in `VoteModal.js` in order to handle different parameters involved. In this case, if the algorithm is chosen as Range, then the `vote_range` function is called.
```
if(currentElectionDetails?.info?.algorithm == "Range"){
    await CurrentElection.vote_range(id,val).send({ from: account });
}
else if(currentElectionDetails?.info?.algorithm == "General"){
    await CurrentElection.vote_general(candidateId).send({ from: account });
}
```
The required functions for different algorithms can be written in the smart contract. Since solidity doesnot support switch statements, we can use if statements to check the rating given. A mapping can be useful to keep a track of the candidate ids that a voter has already voted. This will be helpful to ensure the voter does not vote again.
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
* Take two parameters: one who has been voted and second what rank it received.
* We need to check if the voter has already voted the candidate. This is done using a loop to check if the candidate id is already present in the array. If yes, then the voter has already voted, otherwise not.
* We also need to check the election's status and the voter is allowed to vote only when it is active.
* If all the checks are passed, we push the candidate id into the array and check for the rating value.
* According to the value, the count gets added up to the candidate's voteCount.

At last, the results for the Range Voting are determined by the exact same process of the General Voting System.
#### 1.1.3 Demonstration video: https://youtu.be/pBfxEnaYEfw
### 1.2 Borda Voting Algorihtm
The Borda algorithm is very similar to Range Voting Algorithm except the fact that Borda algorithm does not allow same rating to be given to multiple candidates.
#### 1.2.1 Implementation Details
The Borda algorithm requires an additional check to ensure that the rating is unique for every candidate. This can be achieved using an array which can store the ratings. Any rating input must be checked using a require statement at first.

## 2. Authentication and user KYC

Elections are very critical for an organization, and anyone with a wallet and few tokens in it should not be allowed to vote until they are KYC verified. But in order to increase the user engagement, we can give the organiser the right to choose if he wants the voters to be KYC verified or not. 

**Why KYC verification should not be mandatory?**
For example, when there is an art contest in Instagram, the one who gets maximum likes on their post wins. In that case, there's no restriction to get people from all over the world to like it. Similarly, if the election is held through our DApp, people can circulate the election link to get as many people onboard to vote for them. This will definitely let more people know about our DApp.

#### 2.1 Implementation Details for KYC verification
We need to capture customer information before onboarding them for voting. This process will require 3 main steps:
* Upload documents for verification
* Admins to verify the documents
* User getting verified to vote

The best way to upload documents in a secured manner is by using IPFS. The InterPlanetary File System (IPFS) is a peer to peer file- sharing protocol connecting computing devices for sharing/ storing files. The below image shows a glimpse of what IPFS does.
![](https://i.imgur.com/Oj3I679.png)

We can use IPFS network to upload and retrieve KYC docs ensuring end to end encryption. Before sharing out the KYC docs into the IPFS network, we consider encrypting the file for extra security and reducing the file size. Since anyone  access the KYC docs from the IPFS network by just knowing their hash values.
I'm proposing this idea from [this](https://ieeexplore.ieee.org/document/9230987) research paper. 

There are three different approaches which can suit well for our Election DApp.

#### 1. When there's a single admin
Digital signatures are used to prove the documents without exposing their private keys. In our application, when the user uploads the document, a encryption function is run in the backend which takes the public key of the admin and the document and encrypts it. While on the admin side, during verification, the encrypted hash can be decrypted using the admin's private key and hash generated. The admin can then access the document and verifies it. I'll be referring to this detailed [article](https://medium.com/mycrypto/the-magic-of-digital-signatures-on-ethereum-98fe184dc9c7) on the medium website.

But there is huge human dependency in the above mentioned process. The process can be explained using a flow diagram:
![](https://i.imgur.com/qtcQyz5.png)

To reduce that, we can apply a Machine Learning algorithm at first which uses image recognition and compares the face capture and the document given. 

#### 2. When there are multiple admins
In this case, whenever a user requests for KYC verification and uploads their document, this document can be stored on IPFS network as discussed above. The idea to have multiple admins is to create a decentralised environment in case one admin is bribed for conducting the elections. When the user gets approved by 50% of the admins, he/she gets KYC verified and the right to vote in the election. 
#### 3. If voter ids are available on a public database
In current scenario, there's an API through which we can check if an Aadhar card is valid or not. If we find a similar kind of trusted database them we only need to check their voter id along with their public address.
Ideas have also been taken from [this](https://github.com/amand1996/Votechain).

## 3. Open and Invite based elections
The idea is to provide two types of election: open and invite-based. Open elections are elections in which anyone can vote provided that they are KYC verified and invite based elections allows the organisers to send the voters email invites with a secure single-use link. Using that link a voter can vote without KYC verification.
#### 3.1 Implementation Details
While creating an election, the organiser can choose the type of election he/she wants. If he/she chooses the invite-based election, then a second box is popped up to enter the email and submit for sending the link to the voter. 

We can use [MailJS](https://www.emailjs.com/), and use [SendGrid](https://github.com/sendgrid/sendgrid-nodejs) for transactional services. From the front end side, we can send a request to an SMTP server, which will send the email.

A check can be put to see if the election is open or invite-based. If it is open, then the voters have to be KYC verified and in case of invite-based election, we remove the KYC verification check to allow them to vote without being verified.

## 4. Extra features

* **Restrict Dates** - In the present scenerio, an election can be created at any date(even if passed), which will ultimately not allow the organiser to add candidates. An election can only be created after the current date and time. 
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
* **Settings button** -
    * Logout - As of now, we cannot disconnect Metamask using Ethereum API. Reference - [here](https://stackoverflow.com/questions/67856902/how-to-logout-of-metamask-account-using-web3-js)
    So, if the user clicks the logout button, then we can show them the procedure to disconnect the wallet.
    * KYC verification - Fom this button, the user will be able to start its KYC verification.
    * Profile edit - A function in Solidity can be written to update the name of the user.
* **Improve the UI** - The below mentioned image shows that the hovering text is not visible.
![](https://i.imgur.com/JxbwJhV.png)
* **Shorten the visibility of contract address** - The full contract address can make a user uncomfortable during voting. We can show few first and last digits, like `0xaD37....D081`. This can be done using the following code:
```
Wallet: {infoText.slice(0, 6)}...{infoText.slice(-4)}
```
* **Switch Network Functionality** - Currently, if the user is not connected to the `Avalanche Fuji Chain`, the public address does not show up and there's no way the user would understand that he might be in the wrong network. 
# II. Schedule of Deliverables

![](https://i.imgur.com/rMBLpuO.jpg)


### 0. Before Community Bonding

* Get familiar with the Agora Blockchain codebase.
* Be ready with plan and ideas to discuss

### 1. Community Bonding Period
#### 1.1 Week 1:
* Get in touch with my project mentor, introduce myself in the community, workout a good time to communicate (taking into account timezone differences and their schedule).
* Ask the mentor for his ideas/plans for the project. The project proposal reflects my knowledge as of writing, there may be essential amendments required and it would be better to make them early.
#### 1.2 Week 2:
* Go through the Agora Blockchain codebase thoroughly. Make notes of the functions/files that might be helpful in future.
* Explore the existing solutions and keep a track of best approaches found.
* Get in contact with other GSoCer's , both within the AOSSIE community and outside.
#### 1.3 Week 3:
* Discuss the current approach with the mentor and start noting down the advantages and disadvantages of them.
* Come up with the possible approaches and finalize what all features are required.
* Start laying out a general outline of the possible cases.
### 2. Phase 1
#### 2.1 Week 4:
* Start with the algorithms that were finalised.
* Note the  essential attributes, functions, data structures required.
* Layout a basic outline explaining the algorithm.
* Update the UI.
* Write functions in Solidity to implement the logic behind each voting algorithm.
* Unit testing of every function needs to be done.
#### 2.2 Week 5:
* Integrate the front-end with the smart contract.
* Create separate smart contracts to avoid dependencies and it should be loosely coupled.
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
* Integrate the encryption, decryption, verification processes.
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
* Deploying on the testnet.
* Complete any left-over work.
#### 3.3 Week 12:
* Work on the extra features like change of CSS properties, improve the UI.
* Implement the settings button.
* Ogranisers must be allowed to extend the election dates.
#### 3.4 Week 13:
* Testing each function and asserting it to check errors.
* Checking the application with different test cases.
* Finalize and complete the documentation.
* Refactor Code.
#### 3.5 Week 14:
* Deploying on the Avalanche Fuji Chain
* Checking the entire application thoroughly
* Work on left over stuff.
#### 3.6 Week 15:
* Buffer week for unexpected delays, final submissions for GSoC 2022.


# III. Skills
### Programming Skills
>* Fluent in C++, Python
>* Good knowledge of HTML, CSS, JS, Reactjs
>* Solidity
### Blockchain Skills
>* Good knowledge of Blockchain basics including Solidity, Metamask, Web3, etc.
>* Involved in Blockchain specialisation course by Buffalo University
>* Involved in finding various Blockchain applications and knowing them better
>*  Comfortable with Git and GitHub
# IV. Previous Blockchain Project Development Experience
* CrowdFunder - https://github.com/kirtilodha/CrowdFunder

A decentralized platform which allows developers all over the globe to gather funding for their projects. They can request for a campaign and provide details about their project. In order to maintain transparency, developers can spend money only if 50% of the contributors approve.
**Implementation video**: [Link](https://github.com/kirtilodha/CrowdFunder/blob/master/README.md)

* Voting Name Service - https://github.com/kirtilodha/Voting-Name-Service

This is part of a Election Dapp which uses a ".voter" to signup for the dapp. A voter can earn a NFT while registering and minting a domain for their name.

**Implementation**: http://voting-name-service.vercel.app/

* MintNFTs - https://github.com/kirtilodha/MintNFTs

A platform to mint nfts and these are accessible in OpenSea.

**Implementation**: https://mint-nfts-cyan.vercel.app/

# V. Other Open-Source development experience
* **Kharagpur Winter of Code** - https://github.com/kirtilodha/ElectionDapp
    * The project was based on an Election DApp.
    * I contributed to the front-end as well as back-end side.
    **Merged PR**:
https://github.com/Shikhar15606/ElectionDapp/pull/136
* **Hacktoberfest** - Mattermost project - https://github.com/kirtilodha/focalboard
• Created issues 
• Enhanced their webpage
• Resolved 3 issues regarding interaction with the webpage
**Merged PR**
    * https://github.com/mattermost/focalboard/issues/1602
    
    **PRs**
    *   https://github.com/mattermost/focalboard/pull/1670
    *    https://github.com/mattermost/focalboard/pull/1649
    
    
* **GsSOC** – Jagrati WebApp - https://github.com/kirtilodha/JagratiWebApp
• Created issues
• Resolved the existing issues
# VI. Why this Project?
The project is based on Election DApp upon which I've previously worked on. I'm very familiar with the tech stack it uses - Reactjs, Solidity, Ganache, Metamask, Web3. When I looked around the codebase and was integrating the Range Voting Algorithm, I was sure that I can integrate the required features and bring a lot more interesting features onto the table. 
* **Different algorithms:** I found this feature very fascinating and informative. When I implemented the Range Voting algorithm, it created more curiosity to implement the other algorithms.
* **Flexibility and User experience:** Due to its open and invite based feature, the DApp can be used for several different types of purposes.
* **A chance to be a part of an open source commmunity:** I want to take up this opportunity to work in a large scale open source organisation.
# VII. Do you have any commitments during the GSoC Period?
I have no commitments for this summer and will be available to contribute to the project.
# VIII. About Me
I'm Kirti Lodha currently in 3rd year pursuing Electrical and Electronics engineering from 
DSCE, Bangalore. I'm currently working as a GitHub Extern'22, in which the project is based on Metaverse. I also have been selected as a MLSA in April 2020. I was among the top 12 
selected in WIEHACK 3.0 hackathon. Hackathons, innovations, projects, webinars are my 
favourite kinds of work. 
I’m a Blockchain enthusiast and a competitive programmer. I code in C++ and have good 
knowledge on HTML, CSS, JS, Reactjs. I’ve been coding in Solidity from a year now, which is based on C++ 
and JS. Earlier I’ve worked on research papers with a professor from my university, DSCE.
# IX. Why Me?
I've extensively used Solidity language and Reactjs to make personal projects and explored the field in data privacy, secured transactions, contract creation. 
I've maintained proper documentation for every project for easier understanding in the upcoming times.

I'm fairly confident to work on the project and make it a success.


---

### Thank You






