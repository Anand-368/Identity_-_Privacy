// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title Decentralized Identity Verification System
 * @dev A privacy-focused smart contract for identity management and verification
 * @author Identity & Privacy Project Team
 */
contract Project {
    
    // Events
    event IdentityRegistered(address indexed user, bytes32 indexed identityHash, uint256 timestamp);
    event IdentityVerified(address indexed user, address indexed verifier, uint256 timestamp);
    event VerificationRevoked(address indexed user, address indexed verifier, uint256 timestamp);
    
    // Structs
    struct Identity {
        bytes32 identityHash;        // Hash of user's identity data (for privacy)
        uint256 registrationTime;   // When identity was registered
        uint256 verificationCount;  // Number of verifications received
        bool isActive;              // Whether identity is active
        mapping(address => bool) verifiers; // Authorized verifiers for this identity
    }
    
    struct Verifier {
        bool isAuthorized;          // Whether verifier is authorized
        uint256 verificationsGiven; // Total verifications given by this verifier
        string verifierType;        // Type of verifier (e.g., "KYC", "Academic", "Professional")
    }
    
    // State Variables
    mapping(address => Identity) private identities;
    mapping(address => Verifier) public verifiers;
    address public owner;
    uint256 public totalIdentities;
    uint256 public totalVerifiers;
    
    // Modifiers
    modifier onlyOwner() {
        require(msg.sender == owner, "Only contract owner can perform this action");
        _;
    }
    
    modifier onlyAuthorizedVerifier() {
        require(verifiers[msg.sender].isAuthorized, "Only authorized verifiers can perform this action");
        _;
    }
    
    modifier identityExists(address user) {
        require(identities[user].isActive, "Identity does not exist or is inactive");
        _;
    }
    
    // Constructor
    constructor() {
        owner = msg.sender;
        totalIdentities = 0;
        totalVerifiers = 0;
    }
    
    /**
     * @dev Core Function 1: Register a new identity with privacy-preserving hash
     * @param _identityHash Hash of the user's identity data (computed off-chain)
     * Users can register their identity by providing a hash of their personal data
     * This ensures privacy while allowing verification
     */
    function registerIdentity(bytes32 _identityHash) external {
        require(!identities[msg.sender].isActive, "Identity already registered");
        require(_identityHash != bytes32(0), "Invalid identity hash");
        
        Identity storage newIdentity = identities[msg.sender];
        newIdentity.identityHash = _identityHash;
        newIdentity.registrationTime = block.timestamp;
        newIdentity.verificationCount = 0;
        newIdentity.isActive = true;
        
        totalIdentities++;
        
        emit IdentityRegistered(msg.sender, _identityHash, block.timestamp);
    }
    
    /**
     * @dev Core Function 2: Verify an identity (only authorized verifiers)
     * @param _userAddress Address of the user whose identity is being verified
     * @param _proofHash Hash of the verification proof provided by user
     * Authorized verifiers can verify identities by checking proof against registered hash
     */
    function verifyIdentity(address _userAddress, bytes32 _proofHash) external 
        onlyAuthorizedVerifier 
        identityExists(_userAddress) 
    {
        require(_proofHash != bytes32(0), "Invalid proof hash");
        require(
            identities[_userAddress].identityHash == _proofHash, 
            "Proof does not match registered identity hash"
        );
        require(
            !identities[_userAddress].verifiers[msg.sender], 
            "Identity already verified by this verifier"
        );
        
        identities[_userAddress].verifiers[msg.sender] = true;
        identities[_userAddress].verificationCount++;
        verifiers[msg.sender].verificationsGiven++;
        
        emit IdentityVerified(_userAddress, msg.sender, block.timestamp);
    }
    
    
    function checkVerificationStatus(address _userAddress, address _verifierAddress) 
        external 
        view 
        identityExists(_userAddress)
        returns (bool isVerified, uint256 totalVerifications, uint256 registrationTime) 
    {
        Identity storage userIdentity = identities[_userAddress];
        
        isVerified = userIdentity.verifiers[_verifierAddress];
        totalVerifications = userIdentity.verificationCount;
        registrationTime = userIdentity.registrationTime;
        
        return (isVerified, totalVerifications, registrationTime);
    }
    
    // Additional utility functions for contract management
    
    /**
     * @dev Add authorized verifier (only owner)
     * @param _verifier Address of the verifier to authorize
     * @param _verifierType Type of verification they can perform
     */
    function addVerifier(address _verifier, string memory _verifierType) external onlyOwner {
        require(_verifier != address(0), "Invalid verifier address");
        require(!verifiers[_verifier].isAuthorized, "Verifier already authorized");
        
        verifiers[_verifier] = Verifier({
            isAuthorized: true,
            verificationsGiven: 0,
            verifierType: _verifierType
        });
        
        totalVerifiers++;
    }
    
    /**
     * @dev Revoke verification (only the verifier who gave it)
     * @param _userAddress User whose verification to revoke
     */
    function revokeVerification(address _userAddress) external 
        onlyAuthorizedVerifier 
        identityExists(_userAddress) 
    {
        require(
            identities[_userAddress].verifiers[msg.sender], 
            "No verification to revoke"
        );
        
        identities[_userAddress].verifiers[msg.sender] = false;
        identities[_userAddress].verificationCount--;
        verifiers[msg.sender].verificationsGiven--;
        
        emit VerificationRevoked(_userAddress, msg.sender, block.timestamp);
    }
    
    /**
     * @dev Get verifier information
     * @param _verifier Address of the verifier
     */
    function getVerifierInfo(address _verifier) external view 
        returns (bool isAuthorized, uint256 verificationsGiven, string memory verifierType) 
    {
        Verifier storage verifier = verifiers[_verifier];
        return (verifier.isAuthorized, verifier.verificationsGiven, verifier.verifierType);
    }
    
    /**
     * @dev Check if an identity is registered (privacy-safe)
     * @param _userAddress Address to check
     */
    function isIdentityRegistered(address _userAddress) external view returns (bool) {
        return identities[_userAddress].isActive;
    }
}
