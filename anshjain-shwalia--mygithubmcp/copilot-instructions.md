## mygithubmcp

> // Package models defines the core data structures used throughout the application for representing


**github_models.go**


// Package models defines the core data structures used throughout the application for representing
// GitHub entities, API requests, and responses. These models are designed to be concise and efficient for API usage.
package models

import (
	"encoding/json"
	"fmt"
	"time"
)

// ConciseIssue represents a minimal, essential view of a GitHub issue for API responses.
// It omits verbose fields to save bandwidth and context space.
type ConciseIssue struct {
	ID            int64     `json:"id"`           // Unique GitHub issue ID
	Number        int       `json:"number"`       // Issue number within the repository
	CreatedAt     time.Time `json:"created_at"`   // Creation timestamp
	Title         string    `json:"title"`        // Issue title
	State         string    `json:"state"`        // Issue state (open, closed, etc.)
	Labels        []string  `json:"labels"`       // List of label names
	CommentsCount int       `json:"comments_count"`// Number of comments on the issue
}

// ConciseRepo represents a minimal view of a GitHub repository for search and listing APIs.
type ConciseRepo struct {
	ID          int64  `json:"id"`                // Unique repository ID
	FullName    string `json:"full_name"`         // Full name (owner/repo)
	Description string `json:"description"`       // Repository description
	URL         string `json:"html_url"`          // HTML URL to the repository
	Stars       int    `json:"stargazers_count"`  // Number of stars
	Forks       int    `json:"forks_count"`       // Number of forks
	OpenIssues  int    `json:"open_issues_count"` // Number of open issues
}

// SearchRepoParams defines the parameters for searching repositories.
type SearchRepoParams struct {
	Query string `json:"query"` // The search query string
}

// User represents a GitHub user or organization in a simplified form.
type User struct {
	ID        int64  `json:"id"`         // Unique GitHub user/org ID
	Login     string `json:"login"`      // Username/login
	Name      string `json:"name"`       // Display name
	Email     string `json:"email"`      // Email address
	AvatarURL string `json:"avatar_url"` // Avatar image URL
	URL       string `json:"html_url"`   // Profile HTML URL
}

// RepoDetailsParams specifies the owner and repository name for repository detail queries.
type RepoDetailsParams struct {
	Owner string `json:"owner"` // Repository owner
	Repo  string `json:"repo"`  // Repository name
}

// SearchIssuesParams defines parameters for searching issues within a repository.
type SearchIssuesParams struct {
	Owner   string        `json:"owner"`           // Repository owner
	Repo    string        `json:"repo"`            // Repository name
	Filters *IssueFilters `json:"filters,omitempty"`// Optional filters (state, labels)
}

// IssueFilters provides optional filters for issue search, such as state and labels.
type IssueFilters struct {
	State  string   `json:"state,omitempty"`  // Issue state to filter (open, closed, all)
	Labels []string `json:"labels,omitempty"` // List of label names to filter
}

// StringSliceOrString is a custom type that supports unmarshaling from either a single string or an array of strings in JSON.
// Useful for flexible API inputs.
type StringSliceOrString []string

// UnmarshalJSON implements custom unmarshaling logic to allow StringSliceOrString to accept either a single string or a slice of strings.
func (s *StringSliceOrString) UnmarshalJSON(data []byte) error {
	// Try to unmarshal as a single string
	var single string
	if err := json.Unmarshal(data, &single); err == nil {
		*s = StringSliceOrString{single}
		return nil
	}
	// If not a string, try to unmarshal as a slice of strings
	var arr []string
	if err := json.Unmarshal(data, &arr); err == nil {
		*s = arr
		return nil
	}
	// If neither, return an error
	return fmt.Errorf("StringSliceOrString: value is neither string nor []string: %s", string(data))
}

// ToSlice returns the value as a slice of strings.
func (s StringSliceOrString) ToSlice() []string {
	return []string(s)
}

// CreateIssueParams defines parameters for creating a new GitHub issue.
type CreateIssueParams struct {
	Owner     string              `json:"owner"`           // Repository owner
	Repo      string              `json:"repo"`            // Repository name
	Title     string              `json:"title"`           // Issue title
	Body      string              `json:"body,omitempty"`  // Optional issue body
	Labels    StringSliceOrString `json:"labels,omitempty"`// Optional labels
	Assignees []string            `json:"assignees,omitempty"`// Optional assignees
}

// CreateRepoRequest defines the parameters for creating a new GitHub repository.
type CreateRepoRequest struct {
	Name        string `json:"name"`                // Repository name
	Description string `json:"description,omitempty"`// Optional description
	Private     bool   `json:"private,omitempty"`   // Whether repository is private
	AutoInit    bool   `json:"auto_init,omitempty"` // Whether to initialize with a README
}

// CreateRepoResponse represents the essential details of a newly created repository.
type CreateRepoResponse struct {
	Name     string `json:"name"`      // Repository name
	FullName string `json:"full_name"` // Full name (owner/repo)
	HTMLURL  string `json:"html_url"`  // HTML URL to the repository
}

// CreateRepoParams wraps CreateRepoRequest for API compatibility.
type CreateRepoParams struct {
	CreateRepoRequest
}

// CloseIssueParams defines parameters for closing an issue, including an optional closing comment.
type CloseIssueParams struct {
	Owner       string `json:"owner"`           // Repository owner
	Repo        string `json:"repo"`            // Repository name
	IssueNumber int    `json:"issue_number"`    // Issue number
	Comment     string `json:"comment,omitempty"`// Optional closing comment
}

// IssueParams specifies the repository and issue number for issue queries.
type IssueParams struct {
	Owner       string `json:"owner"`        // Repository owner
	Repo        string `json:"repo"`         // Repository name
	IssueNumber int    `json:"issue_number"` // Issue number
}

// New model structs for the requested functions

// Comments related models
// IssueCommentsParams specifies parameters for retrieving comments on a specific issue.
type IssueCommentsParams struct {
	Owner       string `json:"owner"`        // Repository owner
	Repo        string `json:"repo"`         // Repository name
	IssueNumber int    `json:"issue_number"` // Issue number
}

// ConciseComment represents a minimal view of a GitHub comment, used for both issues and pull requests.
type ConciseComment struct {
	ID        int64     `json:"id"`         // Comment ID
	Body      string    `json:"body"`       // Comment body text
	CreatedAt time.Time `json:"created_at"` // Creation timestamp
	User      *User     `json:"user"`       // Author of the comment
}

// Pull request related models
// PullRequestParams specifies the repository and pull request number for PR queries.
type PullRequestParams struct {
	Owner      string `json:"owner"`      // Repository owner
	Repo       string `json:"repo"`       // Repository name
	PullNumber int    `json:"pull_number"`// Pull request number
}

// ListPullRequestsParams defines parameters for listing pull requests in a repository.
type ListPullRequestsParams struct {
	Owner   string              `json:"owner"`           // Repository owner
	Repo    string              `json:"repo"`            // Repository name
	Filters *PullRequestFilters `json:"filters,omitempty"`// Optional filters (state, sort)
}

// PullRequestFilters provides optional filters for pull request listing.
type PullRequestFilters struct {
	State string `json:"state,omitempty"` // PR state to filter (open, closed, all)
	Sort  string `json:"sort,omitempty"`  // Sort option (created, updated, etc.)
}

// ConcisePullRequest represents a minimal view of a pull request for API responses.
type ConcisePullRequest struct {
	ID            int64     `json:"id"`            // Pull request ID
	Number        int       `json:"number"`        // Pull request number
	Title         string    `json:"title"`         // Title of the pull request
	State         string    `json:"state"`         // State (open, closed, merged, etc.)
	CreatedAt     time.Time `json:"created_at"`    // Creation timestamp
	UpdatedAt     time.Time `json:"updated_at"`    // Last updated timestamp
	User          *User     `json:"user"`          // Author of the pull request
	BaseBranch    string    `json:"base_branch"`   // Base branch name
	HeadBranch    string    `json:"head_branch"`   // Head branch name
	IsDraft       bool      `json:"is_draft"`      // Whether the PR is a draft
	CommentsCount int       `json:"comments_count"`// Number of comments
	ReviewsCount  int       `json:"reviews_count"` // Number of reviews
}

// ConciseReview represents a minimal view of a pull request review.
type ConciseReview struct {
	ID        int64     `json:"id"`         // Review ID
	State     string    `json:"state"`      // Review state (APPROVED, CHANGES_REQUESTED, etc.)
	Body      string    `json:"body,omitempty"` // Optional review body
	User      *User     `json:"user"`       // Reviewer
	CreatedAt time.Time `json:"created_at"` // Review submission timestamp
}

// Branch related model
// ListBranchesParams specifies parameters for listing branches in a repository.
type ListBranchesParams struct {
	Owner string `json:"owner"` // Repository owner
	Repo  string `json:"repo"`  // Repository name
}

// ConciseBranch represents a minimal view of a repository branch.
type ConciseBranch struct {
	Name      string `json:"name"`        // Branch name
	Protected bool   `json:"protected"`   // Whether branch is protected
	Commit    string `json:"commit_sha"`  // SHA of the head commit
}

// Issue timeline model
// IssueTimelineParams specifies parameters for retrieving an issue's timeline of events.
type IssueTimelineParams struct {
	Owner       string `json:"owner"`        // Repository owner
	Repo        string `json:"repo"`         // Repository name
	IssueNumber int    `json:"issue_number"` // Issue number
}

// TimelineEvent represents an event in the timeline of a GitHub issue (e.g., comment, label change, assignment).
type TimelineEvent struct {
	ID        int64     `json:"id"`            // Event ID
	Event     string    `json:"event"`         // Event type (e.g., labeled, assigned, commented)
	CreatedAt time.Time `json:"created_at"`    // Event timestamp
	Actor     *User     `json:"actor,omitempty"`// Actor who performed the event
	Body      string    `json:"body,omitempty"` // Optional body (for comments, etc.)
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AnshJain-Shwalia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
